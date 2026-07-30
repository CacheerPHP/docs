# Tiered caching (L1/L2)

Put a fast local store in front of a shared one. Reads hit local memory first and
fall through to the network only on a miss; the value is promoted back to L1.

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Kernel\Ttl;
use Silviooosilva\CacheerPhp\Stores\ArrayStore;
use Silviooosilva\CacheerPhp\Stores\Support\PredisConnection;
use Silviooosilva\CacheerPhp\Stores\RedisStore;
use Silviooosilva\CacheerPhp\Support\SystemClock;

$clock = new SystemClock();
$l2 = new RedisStore(new PredisConnection(new Predis\Client()), 'app', clock: $clock);

$cache = Cacheer::tiered(
    l1: new ArrayStore($clock),  // per-process, instant
    l2: $l2,                     // shared across the fleet
    l1MaxTtl: Ttl::seconds(10),  // cap how long a value may live locally
);

$user = $cache->remember('user:42', '10 minutes', fn () => $users->find(42));
// first call: miss in both, computed, written to L2 + L1
// next calls in this process: served from L1, no Redis round-trip
```

- Writes and deletes go to **both** layers, so they can't drift.
- A shared generation token keeps long-running workers coherent when another
  worker invalidates L2.
- Tiering optimizes **speed**, not availability — an L2 miss is a real miss. For
  outages, see the [Resilient store](../guides/resilient-store.md).

See the [Tiered caching guide](../guides/tiered-caching.md).
