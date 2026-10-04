# Tiered caching (L1/L2)

A tiered cache puts a **fast local store (L1)** in front of a **shared store (L2)**.
Reads hit local memory first and only fall through to the network on a miss; the
value is then promoted back into L1 so the next read is fast.

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Stores\ArrayStore;

$cache = Cacheer::tiered(
    l1: new ArrayStore($clock),   // per-process, nanosecond reads
    l2: $redisStore,              // shared across the fleet
);
```

## Read-through and promotion

1. `get()` checks L1. Hit → return it.
2. Miss → check L2. Hit → **promote** into L1, then return it.
3. Miss in both → a real miss.

Subsequent reads in the same process are served from L1 without touching L2.
A promoted copy keeps the value's original creation time and absolute expiry, so
its age — and therefore `flexible()` freshness — does not restart when it moves
into L1, even when L1's TTL is capped.

## Writes, deletes, clears

Writes and deletes go to **both** layers so they can't drift:

- `set()` writes L2 and L1 (L1's TTL is capped — see below).
- `delete()` removes from both.
- `clear()` clears both.

L2 is written first, so if it fails the write fails and nothing lands only in L1.

## Capping L1 TTL

L1 is a hot mirror, not the source of truth. A value lives locally for at most
**60 seconds** by default, so a long-lived L2 entry doesn't get pinned in stale
local memory. Tighten or loosen that bound with `l1MaxTtl`, or pass
`Ttl::forever()` to opt out:

```php
use Silviooosilva\CacheerPhp\Kernel\Ttl;

$cache = Cacheer::tiered($l1, $l2, l1MaxTtl: Ttl::seconds(10));
```

## Cross-worker coherence

L1 lives inside a single worker, so when another worker invalidates data in L2,
this worker's L1 could still hold a stale copy. `TieredStore` uses a shared
**generation token**: bulk invalidations bump the generation, and an L1 entry from
an older generation is treated as a miss. Long-running workers therefore pick up
invalidations without restarting.

A **single-key** write or delete by another worker does not bump the generation, so
this worker may keep serving its L1 copy until that copy lapses — at most
`l1MaxTtl` (60 seconds by default). That is the L1 staleness bound; lower
`l1MaxTtl` if your data can't tolerate it.

## Tiering is not resilience

A tiered cache optimizes **performance**: an L2 miss is a genuine miss. If you want
to keep serving when a store is **down**, that's a different tool —
[`ResilientStore`](./resilient-store.md). The two compose: a resilient L2 behind a
tiered L1.

## Observability

Promotions are emitted as `cache.promotion` events when you wrap the tiered cache
with instrumentation:

```php
$cache = Cacheer::tiered($l1, $l2, events: $events);
```

See [Observability](./observability.md).
