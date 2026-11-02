# Differences and limitations

Routed stores keep Laravel's behavior; their few differences are in the first section. The rest of the page covers a `cacheer` store, which follows Laravel's cache contracts but differs from Laravel's built-in stores in a few deliberate ways. Each section explains why and what to do instead.

## Routed stores

A routed store behaves as the store it wraps, with these differences:

- `Cache::getStore()` returns the routed store, not the original class. Code that checks for a class, such as `Cache::getStore() instanceof RedisStore`, should check `Cache::getStore()->native()` instead. Laravel's own checks of this kind are covered: the stores they look for are not routed.
- Writes that go through CacheerPHP report success as `true` or `false`. A custom store that returns something else from `put()`, `putMany()`, `forever()`, `forget()`, or `flush()` gets a boolean back.
- Keys CacheerPHP cannot represent (empty, longer than 1024 bytes, or containing control characters) and lifetimes of zero or less go straight to the store, and are not reported to Cacheer Monitor.
- Cacheer Monitor shows every routed store as `LaravelStore`, so it cannot yet tell a Redis store from a file store. Stores that are not routed are not reported.

## No `Cache::tags()`

Laravel's tags and CacheerPHP's tags mean different things. A Laravel tagged cache namespaces keys by their tag set, so `Cache::tags(['users'])->get('a')` and `Cache::get('a')` are different entries. A CacheerPHP tag is an invalidation set attached to an existing key. Mapping one onto the other would break the semantics of both, so `Cache::tags()` throws `BadMethodCallException` on a `cacheer` store.

To invalidate groups of keys, use CacheerPHP's tags or scopes through the injected contract; see [Laravel features](./framework-features.md#cacheerphp-features-beyond-cache).

## Locks cost more round trips

Laravel needs lock owners that another process can release (`restoreLock()`, `forceRelease()`). CacheerPHP's own locks do not expose their owner, so a Laravel lock is a cache record holding the owner, changed under a short CacheerPHP lock. Acquiring one on Redis takes four round trips where Laravel's Redis lock takes one. In exchange, releases are owner-safe on every backend, and locks fail closed during an outage.

If a section of code takes many locks per request, measure it before switching.

## Counters are whole numbers

`Cache::increment('visits', 1.5)` throws `InvalidArgumentException` instead of truncating the amount, and incrementing a value that is not an integer fails without changing it. That includes a numeric string, such as `'5'` stored with `Cache::put()`, which Laravel's Redis and database stores would increment. Store counters as integers, or create them with `increment()`.

## Counters and locks during an outage

While the primary backend is down, its counters and locks are unavailable, even with a [fallback](./configuration.md#failover): a counter or lock held in the fallback would not be shared with other servers.

- Locks are not acquired: `get()` returns `false`.
- `Cache::add()`, `increment()`, and `decrement()` throw `StoreOperationFailedException`.

Laravel's rate limiters use `add()` and `increment()`, so routes behind the `throttle` middleware fail until the primary recovers. If they must keep working, point `cache.limiter` in `config/cache.php` at a store that does not depend on the same backend.

## Keys

Keys are at most 1024 bytes and cannot contain control characters. Longer or invalid keys throw `InvalidKeyException`.

## Redis connections

The redis backend opens its own connection from the settings of the Laravel connection you name. It ignores Laravel's redis `prefix`, `serializer`, and `compression` options, and Redis clusters are not supported. See [configuration](./configuration.md#backends).

## Database transactions

The database backend uses the connection you name in `connection`, and follows it when Laravel reconnects. Cache writes and lock records made inside `DB::transaction()` on that connection belong to the transaction: they are rolled back with it, and other processes see them only once it commits. To keep the cache independent of your transactions, define a second connection to the same database and name it in `connection`.

## Failover is per process

A process that ends during an outage forgets the keys it changed, so under PHP-FPM a value changed or deleted during a Redis outage can be served again after Redis recovers, until it expires. See [configuration](./configuration.md#failover) for mitigations.

## Cached `null`

Laravel's repository treats `null` as a miss, whatever the store. `Cache::put('key', null)` followed by `Cache::get('key', 'default')` returns `'default'`. The injected CacheerPHP contract does distinguish a cached `null` from a miss, through `entry()`.
