# Resilient store (fallback + circuit breaker)

A resilient cache keeps working when your primary store has a bad day. It serves
from a **primary** store and, when the primary starts failing, trips a **circuit
breaker** and serves from a **fallback** — without hammering the broken backend.

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Stores\ArrayStore;

$cache = Cacheer::resilient(
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

$cache = Cacheer::resilient($primary, $fallback, breaker: new CircuitBreaker(/* thresholds */));
```

## Fails closed, never wrong

When the breaker is open and the fallback also misses, the result is a **miss** —
never stale or fabricated data. Resilience buys availability, not a relaxation of
correctness.

## What fails over, and what doesn't

- **Only outages fail over.** A PDO, Redis, or I/O failure counts against the
  breaker and falls back. A bug (`TypeError`, an invalid argument) or bad data (a
  corrupt or oversized payload, a counter overflow) is rethrown as is — failing
  over would only hide it.
- **The primary's result is authoritative.** Writes go to the primary and are
  mirrored to the fallback on a best-effort basis (a fallback outage doesn't fail
  the write). Only while the primary is unavailable does the fallback's result
  stand.
- **Counters and locks never fail over.** `increment()`, `compareAndSwap()`, and
  `lock()` run on the primary alone, because a second, independent counter or lock
  on the fallback would contradict it. While the primary is down they fail closed:
  counters throw `StoreOperationFailedException`, and locks report "not acquired"
  immediately, so `remember()` simply computes without single-flight.
- **Recovery doesn't resurrect old data.** Keys written or deleted on the fallback
  alone during an outage are invalidated on the primary before it serves again, and
  an outage `clearScope()`/`clearTag()` is replayed. After an outage `clear()`, or
  more than 1,000 such writes, the primary is cleared instead. This is tracked per
  process, so each worker reconciles its own outage writes.

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
use Silviooosilva\CacheerPhp\Stores\ResilientStore;

$shared = new ResilientStore($redisStore, new ArrayStore($clock), clock: $clock);
$cache  = Cacheer::tiered(new ArrayStore($clock), $shared, clock: $clock);
```

See [Observability](./observability.md) to emit `cache.failure` events when the
primary trips.
