# Stampede protection & stale-while-revalidate

Two tools for hot keys: `remember()` prevents a dogpile; `flexible()` keeps
responses instant while data refreshes in the background.

## Single-flight `remember()`

When many requests miss the same key at once, only one computes; the rest wait and
read its result.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;

$cache = Cache::file(__DIR__ . '/cache');

$value = $cache->remember('expensive', ttl: '5 minutes', callback: function () {
    return run_expensive_query(); // runs once across concurrent workers
});
```

This needs a store that can lock (all four built-ins). Without one, `remember()`
still works but isn't stampede-proof.

## `flexible()` — stale-while-revalidate

Serve fresh data directly; once it ages past `fresh`, serve the **stale** value
instantly and refresh it once in the background; past `stale`, recompute.

```php
$feed = $cache->flexible('home:feed', fresh: 30, stale: 300, callback: fn () => build_feed());
```

| Age | Behavior |
|---|---|
| 0–30s | serve cached value |
| 30–300s | serve stale immediately + one background refresh |
| >300s | recompute synchronously |

The refresh runs through the deferred executor. Use
`AfterResponseDeferredExecutor` so the user waits for neither the read nor the
refresh:

```php
use Silviooosilva\CacheerPhp\Support\AfterResponseDeferredExecutor;

$cache = new Cache($store, executor: new AfterResponseDeferredExecutor());
```

See [Remember & locks](../guides/remember-and-locks.md) and
[Stale-while-revalidate](../guides/stale-while-revalidate.md).
