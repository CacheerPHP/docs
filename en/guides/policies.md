# Policies

A `CachePolicy` bundles cross-cutting write behavior — a default TTL, TTL jitter,
negative caching, and serve-stale-on-error — and applies it consistently to every
write and `remember()` on a cache. Reads pass straight through.

```php
use Silviooosilva\CacheerPhp\Config\CachePolicy;

$cache = $cache->withPolicy(
    CachePolicy::defaults()
        ->withTtl('10 minutes')
        ->withJitter(0.10)
        ->withNegativeTtl('30 seconds')
        ->withServeStaleOnError('2 minutes'),
);
```

`withPolicy()` returns another `Cacheer` with the policy bound — the policy is
immutable, and each `with*()` returns a new instance.

Because it is the same type, a policy composes with scoping in either order, and
the resulting cache keeps the whole surface (capabilities included):

```php
$cache->in('billing')->withPolicy($policy);   // identical to…
$cache->withPolicy($policy)->in('billing');

$cache->stats()['policy'];                    // true when one is bound
```

## Default TTL — `withTtl()`

Sets the TTL used when a write doesn't specify one. A per-call TTL still wins.

```php
$policy = CachePolicy::defaults()->withTtl('10 minutes');
$cache->set('k', $v);              // stored for 10 minutes
$cache->set('k', $v, ttl: 3600);   // explicit TTL overrides the policy
```

## TTL jitter — `withJitter()`

Spreads expirations so a batch of entries written together doesn't all expire in
the same instant (which would cause a synchronized stampede). Jitter is a fraction
of the TTL, applied via an injectable randomizer so tests stay deterministic.

```php
$policy = CachePolicy::defaults()->withTtl('10 minutes')->withJitter(0.10);
// each write lands somewhere in ~9–10 minutes
```

## Negative caching — `withNegativeTtl()`

Caches "empty" results (e.g. `null`, `[]`) for a **shorter** time than normal, so a
missing record is retried sooner than a populated one — protecting your database
from repeated lookups of things that don't exist, without pinning the absence for
long.

```php
$policy = CachePolicy::defaults()->withTtl('1 hour')->withNegativeTtl('30 seconds');
$cache->remember('user:999', null, fn () => $users->find(999)); // null cached 30s, a hit 1h
```

## Serve-stale-on-error — `withServeStaleOnError()`

Defines a grace window after logical expiry during which, **if a refresh fails**, the
last good value is served instead of propagating the error. This is about
failures, complementing [`flexible()`](./stale-while-revalidate.md), which is about
latency.

```php
$policy = CachePolicy::defaults()->withTtl('5 minutes')->withServeStaleOnError('2 minutes');
```

## What a policy does not touch

`flexible()`'s `$stale` window is an explicit argument, not a default, so jitter
and negative caching never reshape it — a `flexible($k, 30, 300, ...)` window is
300 seconds whatever the policy says.

## Determinism

Policy behavior is fully deterministic under a fake clock and an injected
randomizer, so jitter, negative caching, and grace windows are unit-testable to
the second.
