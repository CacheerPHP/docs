# Laravel features on CacheerPHP

With a `cacheer` store as the default, Laravel's cache-backed features run on CacheerPHP without code changes. This page covers what to expect from each.

## The cache API

`Cache::get()`, `put()`, `many()`, `putMany()`, `remember()`, `forever()`, `forget()`, and the `cache()` helper behave as on Laravel's own stores. A few details:

- `Cache::add()` is atomic across processes: CacheerPHP serializes the check and the write with a lock.
- `Cache::increment()` and `decrement()` start a missing counter at zero. The new counter never expires; an existing one keeps its lifetime. Amounts must be whole numbers.
- `Cache::touch()` extends a live entry's lifetime and never revives an expired one.
- `Cache::flush()` clears only the store's [prefix](./configuration.md#prefix), and leaves held locks held.
- `Cache::putMany()` accepts numeric keys, such as models keyed by ID.
- `Cache::flexible()` uses Laravel's own implementation, which stores its timestamps and refresh lock through the driver.

## Locks

```php
use Illuminate\Support\Facades\Cache;

Cache::lock('reports:monthly', 120)->get(function () {
    // Runs in one process at a time.
});
```

Locks keep Laravel's semantics:

- `$lock->owner()` and `Cache::restoreLock($name, $owner)` let another process release a lock, as queued jobs do.
- `forceRelease()` frees a lock whatever its owner.
- `refresh()` restarts the lifetime, only for the current owner. A lock rebuilt with `restoreLock()` does not know its lifetime, so `refresh()` without seconds keeps the current one.
- A lock with no lifetime (`Cache::lock('name')`) is held until it is released.

Releases are owner-safe: if a holder's lock expires and another process takes it, the first holder can no longer release it. During a backend outage, locks fail closed: `get()` returns `false`. The cost is extra round trips; see [differences and limitations](./limitations.md#locks-cost-more-round-trips).

## Queues and the scheduler

These Laravel features use cache locks and work on a `cacheer` store:

- **Unique jobs** (`ShouldBeUnique`): a duplicate dispatch is dropped while the job is pending, and the lock is released when the job finishes.
- **The scheduler's `withoutOverlapping()`**: a task does not start while its previous run is still going.
- **Isolatable commands**: `php artisan your:command --isolated` exits if another instance is running.

They use the default cache store unless the feature names another one, for example `uniqueVia()` on a job.

## Rate limiting

`RateLimiter::attempt()`, `hit()`, and `tooManyAttempts()`, and the `throttle` middleware for limiters backed by the cache, count hits with atomic `add()` and `increment()`, so limits hold across workers that share the backend.

While the primary backend is down, `add()` and `increment()` throw, so rate-limited routes fail even with a fallback configured. See [counters and locks during an outage](./limitations.md#counters-and-locks-during-an-outage).

## CacheerPHP features beyond `Cache::`

The driver binds CacheerPHP's `Cache` contract to the cache behind your default `cacheer` store (or the first `cacheer` store). Inject it where you need features `Cache::` cannot express:

```php
use Silviooosilva\CacheerPhp\Contracts\Cache;

final class ProductCatalog
{
    public function __construct(private readonly Cache $cache)
    {
    }

    public function products(): array
    {
        return $this->cache->in('catalog')->remember('products', 300, fn (): array => $this->load());
    }

    public function forgetCategory(string $category): void
    {
        $this->cache->flushTag("category:{$category}");
    }
}
```

See [scopes](../guides/scopes.md), [tags](../tutorials/example-10-tagging.md), and the [cache API](../api/cache-functions.md).

## Monitoring

Install [Cacheer Monitor](../cacheer-monitor/quick-start.md) and every `cacheer` store reports its operations, with no further setup.

## Testing

Entries expire on Laravel's clock, so time travel works as it does with Laravel's stores:

```php
Cache::put('token', 'abc', 60);

$this->travel(61)->seconds();

$this->assertNull(Cache::get('token'));
```

For a store that lives only for the test, use `'backend' => 'array'`, for example in `phpunit.xml`:

```xml
<env name="CACHE_STORE" value="cacheer_testing"/>
```

with a matching store in `config/cache.php`:

```php
'cacheer_testing' => ['driver' => 'cacheer', 'backend' => 'array'],
```
