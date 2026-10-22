# CacheerPHP vs Laravel Cache

Choose based on where your application runs, how you invalidate data, and how much storage control you need. Both provide a readable cache API. CacheerPHP keeps that API independent of the application framework; Laravel Cache is already integrated into Laravel.

This comparison covers **CacheerPHP 6.x** and **Laravel 13.x**, checked on **October 20, 2026**. It compares behavior and integration, without performance rankings.

## When Laravel Cache is the simpler choice

Use Laravel Cache when its existing stores and conventions meet your needs. Your application already has configuration, container bindings, events, and testing tools for it. Read-through caching, stale-while-revalidate, locks, and failover are available there too. See [Laravel's cache documentation](https://laravel.com/docs/13.x/cache).

The [Illuminate Cache component](https://github.com/illuminate/cache/blob/13.x/composer.json) can also be installed separately; Laravel Cache is not restricted to full Laravel applications.

## When to consider CacheerPHP

- **You share application services across PHP projects.** Inject the same `Contracts\Cache` interface in a Laravel application, a vanilla PHP service, or a worker.
- **You want explicit invalidation boundaries.** Clear a catalog or tenant scope independently. Scopes qualify keys, tags, counters, and lock names. The four built-in stores all support tags and scoped clearing.
- **You need to compose storage behavior.** Configure L1/L2 stores, circuit-breaker fallback, TTL policies, and an optional compression/encryption pipeline on explicit instances.
- **You want Cacheer-specific visibility.** Add the optional local Cacheer Monitor dashboard to inspect operations, timings, and key activity. Value capture is opt-in.

## Compare the behavior

| Decision | CacheerPHP 6.x | Laravel Cache 13.x |
| --- | --- | --- |
| Application integration | Construct and inject `Contracts\Cache`; in Laravel, use the `cacheer` cache driver behind `Cache::` or bind it explicitly | Integrated configuration, facade, and container contracts; standalone Illuminate component available |
| Built-in storage | File, array, PDO database, Redis | Also includes Memcached, DynamoDB, and filesystem-disk storage |
| Cache a computed result | `remember()`; coordinates misses when locking is available, within lease/timeout limits | `remember()`; explicit locks are available |
| Serve stale data | `flexible()`; default executor is synchronous; deferred execution is configurable | `flexible()` uses Laravel's deferred response lifecycle |
| Cache tags | Supported by all four built-in stores | Unsupported on file, database, DynamoDB, and storage drivers |
| Clear one logical keyspace | `scope('catalog')->clear()`; root clearing stays within the configured store keyspace | Driver-dependent tags or store separation; `flush()` disregards configured prefixes |
| Local read reuse | `TieredStore` composes L1/L2, preserving expiry and bounding L1 age | `memo()` retains resolved reads for one request or job |
| Backend failure | Circuit-breaker fallback for backend outages; primary counters and locks fail closed | Configurable ordered `failover` store |
| Cached `null` | A valid hit, including in `remember()` and `has()` | `get()`/`remember()` treat null as a miss; `has()` returns false |

Laravel references: [cache features and driver limitations](https://laravel.com/docs/13.x/cache), [repository API](https://api.laravel.com/docs/13.x/Illuminate/Cache/Repository.html), and [null/read-through implementation](https://github.com/laravel/framework/blob/13.x/src/Illuminate/Cache/Repository.php). CacheerPHP details: [stores and capabilities](../api/drivers.md), [scopes](./scopes.md), [tiering](./tiered-caching.md), and [resilience](./resilient-store.md).

## The same application task

Laravel Cache:

```php
use Illuminate\Support\Facades\Cache;

$products = Cache::remember('catalog:products', 300, fn () => $repository->all());
```

CacheerPHP, with a cache supplied by your application's bootstrap or container:

```php
use Silviooosilva\CacheerPhp\Contracts\Cache;

function products(Cache $cache, callable $loadProducts): array
{
    return $cache->scope('catalog')->remember('products', 300, $loadProducts);
}
```

The useful distinction here is how the cache is configured, composed, and invalidated. A similar method name does not make the two systems interchangeable: account for null handling, TTL rules, tags, and lock semantics when migrating. They also use separate storage layouts; recompute cached values rather than sharing their raw entries.

## Try the path that fits

- [Run a complete CacheerPHP example](../getting-started/index.md): install, observe a miss and a hit, then invalidate a result.
- [Run Laravel's cache on CacheerPHP](../laravel/index.md): the `cacheer` driver keeps `Cache::`, locks, queues, and the scheduler, with CacheerPHP underneath.
- [Bind CacheerPHP in Laravel services](./laravel.md): a provider binding and an injected catalog service, leaving Laravel's cache as it is.
- [Connect Cacheer Monitor](../cacheer-monitor/quick-start.md): inspect the behavior before adding more cache layers.
- [Keep Laravel's native cache](https://laravel.com/docs/13.x/cache): follow its built-in conventions when they fit your application.
