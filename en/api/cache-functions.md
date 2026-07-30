# Cacheer & ScopedCacheer — method reference

`Silviooosilva\CacheerPhp\Cacheer` is the public entry point. `ScopedCacheer`
(returned by `scope()`) exposes the same read/write surface bound to a scope.
Every key argument accepts a `string` or a [`Key`](./option-builder.md#key).

## Named constructors

Build a `Cacheer` for a store without touching the constructor:

```php
use Silviooosilva\CacheerPhp\Cacheer;

Cacheer::inMemory(?Clock $clock = null): Cache
Cacheer::file(string $directory, ?PipelineConfig $pipeline = null, ?Clock $clock = null): Cache
Cacheer::database(PDO $pdo, string $table = 'cacheer_store', ?PipelineConfig $pipeline = null, ?Clock $clock = null): Cache
Cacheer::redis(RedisConnection $connection, string $prefix = 'cacheer', ?PipelineConfig $pipeline = null, ?Clock $clock = null): Cache

// Decorators
Cacheer::tiered(Store $l1, Store $l2, ?Ttl $l1MaxTtl = null, ?Clock $clock = null, ?DeferredExecutor $executor = null, ?EventDispatcher $events = null): Cache
Cacheer::resilient(Store $primary, Store $fallback, ?CircuitBreaker $breaker = null, ?Clock $clock = null, ?DeferredExecutor $executor = null): Cache
Cacheer::instrumented(Store $store, EventDispatcher $events, bool $captureValues = false, ?callable $redactor = null, ?Clock $clock = null): Cache
```

Or construct directly with any store:

```php
$cache = new Cacheer($store, ?Clock $clock, ?DeferredExecutor $executor, ?EventDispatcher $events);
```

## Reading

### `get()`

```php
public function get(string|Key $key, mixed $default = null): mixed
```

Returns the cached value, or `$default` on a miss. A stored `null`/`false`/`0`/
`''` is a hit and is returned as-is — pass a sentinel default, or use `entry()`,
if you must distinguish a stored `null` from a miss.

```php
$user = $cache->get('user:42');
$flag = $cache->get('feature:x', default: false);
```

### `entry()`

```php
public function entry(string|Key $key): CacheEntry
```

Returns the full [`CacheEntry`](./option-builder.md#cacheentry) — hit/miss,
value, timestamps, and remaining TTL.

```php
$entry = $cache->entry('user:42');
if ($entry->isHit()) {
    $value = $entry->value();
    $ttl   = $entry->remainingTtl($clock); // seconds, or null if forever
}
```

### `has()`

```php
public function has(string|Key $key): bool
```

`true` when a live (unexpired) entry exists.

### `many()`

```php
public function many(iterable $keys, mixed $default = null): array
```

Returns `[key => value]`, using `$default` for misses. Uses the store's native
batch read when it implements [`BatchStore`](./drivers.md#batchstore).

```php
$values = $cache->many(['user:1', 'user:2', 'user:3'], default: null);
```

## Writing

### `set()`

```php
public function set(string|Key $key, mixed $value, Ttl|DateInterval|int|string|null $ttl = null): void
```

Stores a value. The TTL accepts a [`Ttl`](./time-builder.md), an `int` (seconds),
a human string (`'10 minutes'`), a `DateInterval`, or `null` (forever). See
[TTL & Clock](./time-builder.md).

```php
$cache->set('user:42', $user, ttl: '10 minutes');
$cache->set('config', $config, ttl: null); // forever
```

### `setMany()`

```php
public function setMany(iterable $values, Ttl|DateInterval|int|string|null $ttl = null): void
```

Stores `['key' => value, ...]` under one TTL. Uses a native batch/transaction
when the store implements `BatchStore`. Keys must be strings.

### `delete()` / `deleteMany()`

```php
public function delete(string|Key $key): bool
public function deleteMany(iterable $keys): bool
```

`delete()` returns `true` when something was removed. `deleteMany()` returns
`true` only if every key was removed.

### `clear()`

```php
public function clear(): void
```

Removes everything in this cache's keyspace. On a `ScopedCacheer`, `clear()` removes
only that scope (requires [`FlushableScopeStore`](./drivers.md#flushablescopestore)).

## Compute-and-store

### `remember()`

```php
public function remember(string|Key $key, Ttl|DateInterval|int|string|null $ttl, callable $callback): mixed
```

Returns the cached value; on a miss, runs `$callback`, stores the result under
`$ttl`, and returns it. When the store implements
[`LockingStore`](./locks.md), `remember()` is **single-flight**: one caller
computes while the rest wait and read the result (no dogpile). Without locking it
degrades to a plain compute-and-store.

```php
$user = $cache->remember('user:42', '10 minutes', fn () => $users->find(42));
```

### `flexible()` — stale-while-revalidate

```php
public function flexible(string|Key $key, int $fresh, int $stale, callable $callback): mixed
```

Serves a value directly while it is *fresh* (within `$fresh` seconds of creation).
Between `$fresh` and `$stale` it serves the **stale** value immediately and
refreshes it once, in the background, via the deferred executor. Past `$stale`
it recomputes synchronously. Requires `0 < fresh < stale`, and composes with a
hard TTL of `$stale`. See [Stale-while-revalidate](../guides/stale-while-revalidate.md).

```php
$feed = $cache->flexible('feed', fresh: 30, stale: 300, callback: fn () => build_feed());
```

## Scopes and policies

### `scope()`

```php
public function scope(string|Scope $scope): ScopedCacheer
```

Returns a [`ScopedCacheer`](../guides/scopes.md) — an isolated keyspace with the
same API. Scopes nest: `$cache->scope('a')->scope('b')`.

```php
$reports = $cache->scope('reports');
$reports->set('daily', $rows);
$reports->clear(); // clears only the "reports" scope
```

### `withPolicy()`

```php
public function withPolicy(CachePolicy $policy): PolicyCacheer
```

Wraps the cache with a [`CachePolicy`](../guides/policies.md) (default TTL, jitter,
negative caching, serve-stale-on-error). Reads pass through; writes and
`remember()` honor the policy.

```php
$cache = $cache->withPolicy(
    CachePolicy::defaults()->withTtl('10 minutes')->withJitter(0.10)->withServeStaleOnError('2 minutes'),
);
```

## Capabilities beyond the kernel

Atomic counters, tags, touch, prune, and inspection live on the **store**
capability interfaces, not on `Cacheer`. Reach them through the store:

```php
$store->increment(Key::named('visits'));        // AtomicStore
$store->tag(Key::named('p1'), 'products');       // TaggableStore
$removed = $store->clearTag('products');
$store->prune();                                 // PrunableStore
```

See [Stores & capabilities](./drivers.md) for the full list.
