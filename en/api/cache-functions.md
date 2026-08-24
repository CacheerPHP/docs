# Cacheer — method reference

`Silviooosilva\CacheerPhp\Cacheer` is the public entry point, and
`Silviooosilva\CacheerPhp\Contracts\Cache` is the interface it implements — the
one application code should type-hint.

There is **one cache type**. `scope()`, `in()`, and `withPolicy()` all return
another `Cacheer`, so nothing is lost along the way: a scoped cache still takes a
policy, a policy-bound cache still scopes, and both still increment, tag, and
lock. Every key argument accepts a `string` or a [`Key`](./option-builder.md#key).

```php
use Silviooosilva\CacheerPhp\Contracts\Cache;

function buildReport(Cache $cache): string { /* ... */ }

buildReport($cache);                                   // plain
buildReport($cache->in('reports')->withPolicy($p));    // scoped + policy-bound
```

## Named constructors

Build a `Cacheer` for a store without touching the constructor:

```php
use Silviooosilva\CacheerPhp\Cacheer;

Cacheer::inMemory(?Clock $clock = null): Cacheer
Cacheer::file(string $directory, ?PipelineConfig $pipeline = null, ?Clock $clock = null): Cacheer
Cacheer::database(PDO $pdo, string $table = 'cacheer_store', ?PipelineConfig $pipeline = null, ?Clock $clock = null): Cacheer
Cacheer::redis(RedisConnection $connection, string $prefix = 'cacheer', ?PipelineConfig $pipeline = null, ?Clock $clock = null): Cacheer

// Decorators
Cacheer::tiered(Store $l1, Store $l2, ?Ttl $l1MaxTtl = null, ?Clock $clock = null, ?DeferredExecutor $executor = null, ?EventDispatcher $events = null): Cacheer
Cacheer::resilient(Store $primary, Store $fallback, ?CircuitBreaker $breaker = null, ?Clock $clock = null, ?DeferredExecutor $executor = null): Cacheer
Cacheer::instrumented(Store $store, EventDispatcher $events, bool $captureValues = false, ?callable $redactor = null, ?Clock $clock = null): Cacheer
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

### `has()` / `missing()`

```php
public function has(string|Key $key): bool
public function missing(string|Key $key): bool
```

`has()` is `true` when a live (unexpired) entry exists; `missing()` is its
inverse, which reads better in a guard clause.

```php
if ($cache->missing('user:42')) {
    return $this->rebuild();
}
```

### `many()`

```php
public function many(iterable $keys, mixed $default = null): array
```

Returns `[key => value]`, using `$default` for misses. Uses the store's native
batch read when it implements [`BatchStore`](./drivers.md#capability-interfaces).

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
```

### `forever()`

```php
public function forever(string|Key $key, mixed $value): void
```

Stores with no expiry. Equivalent to `set($key, $value, null)`, but says so.

```php
$cache->forever('app:config', $config);
```

### `add()`

```php
public function add(string|Key $key, mixed $value, Ttl|DateInterval|int|string|null $ttl = null): bool
```

Stores **only if the key is absent**, returning `true` when this call was the one
that stored it. When the store can lock, the check and the write are serialized,
so it is a sound first-writer-wins across processes; otherwise it degrades to a
single-process check.

```php
if ($cache->add('import:running', 1, ttl: 300)) {
    // we won — run the import
}
```

A falsy stored value is still a value: `add()` will not overwrite a stored
`false`, `0`, or `''`.

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

### `pull()`

```php
public function pull(string|Key $key, mixed $default = null): mixed
```

Reads and deletes in one call, returning the value the key held (or `$default` on
a miss). Useful for one-shot values — flash messages, single-use tokens, claimed
work items.

```php
$message = $cache->pull('flash:user:42');
```

Like `get()`, it reports a stored `null` as the value it is, not as a miss.

### `clear()`

```php
public function clear(): void
```

Removes everything in this cache's keyspace. On a **scoped** cache it removes only
that scope (requires [`FlushableScopeStore`](./drivers.md#capability-interfaces)).

## Compute-and-store

### `remember()`

```php
public function remember(string|Key $key, Ttl|DateInterval|int|string|null $ttl, callable $callback): mixed
```

Returns the cached value; on a miss, runs `$callback`, stores the result under
`$ttl`, and returns it. When the store can lock, `remember()` is **single-flight**:
one caller computes while the rest wait and read the result (no dogpile). Without
locking it degrades to a plain compute-and-store — it never fails for lack of a
lock, including when the store is wrapped in a decorator.

```php
$user = $cache->remember('user:42', '10 minutes', fn () => $users->find(42));
```

### `rememberForever()`

```php
public function rememberForever(string|Key $key, callable $callback): mixed
```

`remember()` with no expiry.

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

The `$stale` window is your explicit contract, so a bound
[policy](../guides/policies.md) never reshapes it with jitter or negative caching.

## Capabilities

Capabilities are implemented by the **store** and reached on the **cache**, so
this cache's scope is applied for you and one clear exception names anything the
backend cannot do. All four built-in stores support all of them.

### `supports()`

```php
public function supports(string $capability): bool
```

Answers truthfully even through decorators — ask before calling if your backend is
pluggable. Never use `instanceof` on a store for this; see
[Stores & capabilities](./drivers.md).

```php
use Silviooosilva\CacheerPhp\Contracts\AtomicStore;

if ($cache->supports(AtomicStore::class)) {
    $cache->increment('visits');
}
```

### `increment()` / `decrement()`

```php
public function increment(string|Key $key, int $amount = 1, ?int $initial = null, Ttl|DateInterval|int|string|null $ttl = null): int
public function decrement(string|Key $key, int $amount = 1, ?int $initial = null, Ttl|DateInterval|int|string|null $ttl = null): int
```

Atomically adjust a counter and return its new value. With `$initial` set, a
missing key is created as `($initial ± $amount)` and the optional `$ttl` applied;
without it, a missing key is left alone. Requires `AtomicStore`.

```php
$cache->increment('page-views', 1, initial: 0);
$cache->decrement('stock:sku-1', 5, initial: 100);
$cache->increment('rate:user:99', 1, initial: 0, ttl: '1 minute');
```

### `touch()`

```php
public function touch(string|Key $key, Ttl|DateInterval|int|string $ttl): bool
```

Extends an entry's lifetime **without rewriting its value**. `false` when the key
is absent. Requires `TouchStore`. (This is v5's `renewCache()`.)

```php
$cache->touch('session:abc', '1 hour');
```

### `tag()` / `flushTag()`

```php
public function tag(string|Key $key, string ...$tags): void
public function flushTag(string $tag): int
```

Associate a key with tags, then invalidate them in bulk; `flushTag()` returns how
many entries were removed. Tag names are namespaced by scope, so two scopes using
the same tag name do not flush each other. Requires `TaggableStore`.

```php
$cache->tag('product:1', 'products', 'catalog');
$removed = $cache->flushTag('products');
```

### `lock()`

```php
public function lock(string $name, Ttl|DateInterval|int|string $ttl = 60): Lock
```

A named cross-process mutex, namespaced by scope. Requires `LockingStore`. See
[Locks](./locks.md).

```php
$lock = $cache->lock('nightly-import', 300);
if ($lock->acquire()) {
    try { /* ... */ } finally { $lock->release(); }
}
```

### `entries()` / `prune()`

```php
public function entries(): iterable   // requires InspectableStore
public function prune(): int          // requires PrunableStore
```

`entries()` walks the live entries in this cache's scope, with metadata.
`prune()` drops expired entries eagerly and returns how many were removed.

```php
foreach ($cache->in('reports')->entries() as $entry) {
    echo $entry->key()->value(), ' → ', $entry->remainingTtl($clock), "\n";
}
```

## Scopes, policies, and views

### `scope()` / `in()`

```php
public function scope(string|Scope $scope): static
public function in(string|Scope $scope): static      // alias
public function boundScope(): Scope
```

Returns another `Cacheer` bound to an isolated keyspace. Scopes nest, and apply to
**every** operation — counters, tags, and locks included. See
[Scopes](../guides/scopes.md).

```php
$reports = $cache->in('reports');
$reports->set('daily', $rows);
$reports->increment('runs');   // cannot collide with another scope
$reports->clear();             // clears only the "reports" scope
```

### `withPolicy()`

```php
public function withPolicy(CachePolicy $policy): static
```

Binds a [`CachePolicy`](../guides/policies.md) (default TTL, jitter, negative
caching, serve-stale-on-error). Reads pass through; writes and `remember()` honor
it. Order does not matter — `in('x')->withPolicy($p)` and
`withPolicy($p)->in('x')` are equivalent.

```php
$cache = $cache->withPolicy(
    CachePolicy::defaults()->withTtl('10 minutes')->withJitter(0.10)->withServeStaleOnError('2 minutes'),
);
```

### `formatted()`

```php
public function formatted(): FormattedCacheer
```

A read-formatting view: value-returning reads hand back a `CacheDataFormatter`
you can chain `->toJson()` / `->toArray()` / `->toObject()` / `->toString()` on.
Everything else forwards unchanged, so the view keeps the whole surface. Call
`raw()` to step back out.

```php
$json = $cache->formatted()->get('user:1')->toJson();
```

### `stats()`

```php
public function stats(): array
```

What this cache *is* — store, scope, whether a policy is bound, and which
capabilities are really available. Safe to log or expose; it never contains
cached values.

```php
[
    'store'        => 'FileStore',
    'scope'        => 'reports',
    'policy'       => true,
    'capabilities' => ['batch' => true, 'atomic' => true, 'locking' => true, /* ... */],
]
```

### `store()`

```php
public function store(): Store
```

The underlying store. Application code should not need it — every capability is
on the cache, with the scope applied. It exists for store authors, tests, and the
CLI.
