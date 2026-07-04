# Example 18 — Stampede Protection & Stale-While-Revalidate

*New in v5.2.0*

When a popular cache key expires, every in-flight request misses at once and they all run the expensive callback together — a *cache stampede* (or "dogpile"). CacheerPHP now prevents that, and adds a stale-while-revalidate mode so reads rarely block on recomputation at all.

## Stampede-safe `remember()`

`remember()` is unchanged to call — the protection is automatic. On a miss, one request computes under a single-flight lock while the others wait and then read the freshly-stored value.

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useFileDriver();

// Even if 100 requests hit this simultaneously on a cold key,
// computeExpensiveStats() runs exactly once.
$stats = $cache->remember('dashboard:stats', 300, function () {
    return computeExpensiveStats(); // slow query, API call, etc.
});
```

This works on any lockable driver (File, Database, Redis). It also applies to the fluent namespace context:

```php
$report = $cache->in('reports')->remember('latest', 600, fn () => buildReport());
```

## Stale-While-Revalidate with `flexible()`

`flexible()` keeps serving a cached value past its *fresh* horizon while a single worker refreshes it in the background, only blocking once the value is older than the *stale* horizon.

```php
// Fresh for 60s; serve-stale-and-refresh for up to 10 minutes.
$html = $cache->flexible('home', fresh: 60, stale: 600, function () {
    return renderHomepage();
});
```

Lifecycle for a given key:

| Age of value | Behaviour |
|--------------|-----------|
| `< 60s` (fresh) | Served directly. |
| `60s – 600s` (stale) | Served immediately; one request recomputes in the background. |
| `> 600s` (expired) | Recomputed under a single-flight lock. |

The request that wins the refresh lock recomputes inline and returns the fresh value; everyone else in the stale window gets the cached value instantly. The callback never runs more than once at a time.

## Choosing between them

| Use | When |
|-----|------|
| `remember()` | You always want a value no older than the TTL, and can tolerate one request blocking on a miss. |
| `flexible()` | You'd rather serve a slightly stale value than ever block readers — dashboards, homepages, expensive aggregations. |

## See also

- [Cache Functions → remember()](../api/cache-functions.md#remember--get-or-compute-with-ttl)
- [Cache Functions → flexible()](../api/cache-functions.md#flexible--stale-while-revalidate)
- [Distributed Locks](../api/locks.md)
