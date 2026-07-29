# Resilient store (fallback + circuit breaker)

A resilient cache keeps working when your primary store has a bad day. It serves
from a **primary** store and, when the primary starts failing, trips a **circuit
breaker** and serves from a **fallback** — without hammering the broken backend.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Stores\ArrayStore;

$cache = Cache::resilient(
    primary:  $redisStore,          // normally serves everything
    fallback: new ArrayStore($clock), // takes over when Redis is unhealthy
);
```

## The circuit breaker

The breaker has three states:

- **Closed** — normal. Requests go to the primary. Failures are counted.
- **Open** — after too many failures, the breaker opens: requests skip the primary
  and go straight to the fallback, so a slow or down backend doesn't drag every
  request into a timeout.
- **Half-open** — after a cool-down, a probe request is allowed through. Success
  closes the breaker; failure re-opens it.

Tune it by passing your own `CircuitBreaker`:

```php
use Silviooosilva\CacheerPhp\Support\CircuitBreaker;

$cache = Cache::resilient($primary, $fallback, breaker: new CircuitBreaker(/* thresholds */));
```

## Fails closed, never wrong

When the breaker is open and the fallback also misses, the result is a **miss** —
never stale or fabricated data. Resilience buys availability, not a relaxation of
correctness.

## Degraded health, no secrets

You can observe the breaker's state (closed/open/half-open) to drive a health
check or dashboard. The exposed health is the breaker state only — it never leaks
connection strings or credentials.

## Resilience vs. tiering

- [`TieredStore`](./tiered-caching.md) is about **speed**: a primary (L2) miss is a
  real miss, and L1 just makes hits fast.
- `ResilientStore` is about **failure**: a primary *outage* is masked by the
  fallback.

They compose. A common shape is a local L1, a shared L2, and a resilient wrapper so
an L2 outage degrades to the fallback instead of erroring:

```php
$shared = Cache::resilient($redisStore, new ArrayStore($clock));
$cache  = Cache::tiered(new ArrayStore($clock), $shared->/* store */);
```

See [Observability](./observability.md) to emit `cache.failure` events when the
primary trips.
