# Differences and limitations

A `cacheer` store follows Laravel's cache contracts, but a few behaviors differ from Laravel's built-in stores. Each one is deliberate; this page explains why and what to do instead.

## No `Cache::tags()`

Laravel's tags and CacheerPHP's tags mean different things. A Laravel tagged cache namespaces keys by their tag set, so `Cache::tags(['users'])->get('a')` and `Cache::get('a')` are different entries. A CacheerPHP tag is an invalidation set attached to an existing key. Mapping one onto the other would break the semantics of both, so `Cache::tags()` throws `BadMethodCallException` on a `cacheer` store.

To invalidate groups of keys, use CacheerPHP's tags or scopes through the injected contract; see [Laravel features](./framework-features.md#cacheerphp-features-beyond-cache).

## Locks cost more round trips

Laravel needs lock owners that another process can release (`restoreLock()`, `forceRelease()`). CacheerPHP's own locks do not expose their owner, so a Laravel lock is a cache record holding the owner, changed under a short CacheerPHP lock. Acquiring one on Redis takes four round trips where Laravel's Redis lock takes one. In exchange, releases are owner-safe on every backend, and locks fail closed during an outage.

If a section of code takes many locks per request, measure it before switching.

## Counters are whole numbers

`Cache::increment('visits', 1.5)` throws `InvalidArgumentException` instead of truncating the amount, and incrementing a value that is not an integer fails without changing it.

## Keys

Keys are at most 1024 bytes and cannot contain control characters. Longer or invalid keys throw `InvalidKeyException`.

## Redis connections

The redis backend opens its own connection from the settings of the Laravel connection you name. It ignores Laravel's redis `prefix`, `serializer`, and `compression` options, and Redis clusters are not supported. See [configuration](./configuration.md#backends).

## Database connections in long-running workers

The database backend uses the PDO connection of the Laravel connection when the store is first used. In a long-running worker, a connection that Laravel later reconnects is not followed by the store; restart workers after a database failover.

## Failover is per process

A process that ends during an outage forgets the keys it changed, so under PHP-FPM a value changed or deleted during a Redis outage can be served again after Redis recovers, until it expires. See [configuration](./configuration.md#failover) for mitigations.

## Cached `null`

Laravel's repository treats `null` as a miss, whatever the store. `Cache::put('key', null)` followed by `Cache::get('key', 'default')` returns `'default'`. The injected CacheerPHP contract does distinguish a cached `null` from a miss, through `entry()`.

## Upgrading from silviooosilva/cacheerphp-laravel 0.1

The 0.1 package wrapped CacheerPHP 4 and 5 behind its own `Cacheer` facade. `cacheerphp/laravel` replaces it with a cache driver:

- Replace `Cacheer::putCache()` and `Cacheer::getCache()` with `Cache::put()` and `Cache::get()`, or with the injected CacheerPHP contract.
- Move the settings from `config/cacheer.php` into a `cacheer` store in `config/cache.php`.
- The `cacheer:monitor`, `cacheer:events-clear`, and `cacheer:diagnostics` commands are gone. Use [Cacheer Monitor](../cacheer-monitor/index.md) and the [`cacheer` CLI](../guides/cli.md) that ships with CacheerPHP.
- CacheerPHP 6 cannot read data written by CacheerPHP 5. Start with an empty cache, for example under a new `prefix`. See [migrating from v5](../updating/v5-migration.md).
