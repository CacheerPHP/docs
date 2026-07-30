# DateInterval TTL

Any TTL argument accepts a native PHP `DateInterval`.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::inMemory();

$cache->set('a', $v, ttl: new DateInterval('PT30M'));  // 30 minutes
$cache->set('b', $v, ttl: new DateInterval('P1D'));    // 1 day
$cache->set('c', $v, ttl: new DateInterval('PT1H30M')); // 90 minutes

$cache->remember('d', new DateInterval('PT10M'), fn () => compute());
```

Rules:

- Intervals with years or months are rejected (their length is ambiguous) — use
  days/hours/minutes/seconds. A negative interval is rejected too.
- Pin an absolute expiry instead with `Ttl::until()`:

  ```php
  use Silviooosilva\CacheerPhp\Kernel\Ttl;
  use Silviooosilva\CacheerPhp\Support\SystemClock;

  $cache->set('sale', $v, ttl: Ttl::until(new DateTimeImmutable('2026-12-31'), new SystemClock()));
  ```

See [TTL & Clock](../api/time-builder.md).
