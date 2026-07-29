# Custom expiration (TTL)

Every method that stores a value accepts the same TTL forms.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Kernel\Ttl;

$cache = Cache::inMemory();

$cache->set('a', $v, ttl: 3600);                     // int seconds
$cache->set('b', $v, ttl: '2 hours');                // human string
$cache->set('c', $v, ttl: new DateInterval('PT30M')); // DateInterval
$cache->set('d', $v, ttl: Ttl::minutes(15));         // Ttl object
$cache->set('e', $v, ttl: null);                     // forever
$cache->set('f', $v, ttl: 'forever');                // forever (string)
```

Human strings accept `<n> second|minute|hour|day|week` (singular or plural).

## Reading remaining TTL

Use `entry()` to see how long is left:

```php
use Silviooosilva\CacheerPhp\Support\SystemClock;

$entry = $cache->entry('b');
echo $entry->remainingTtl(new SystemClock()) ?? 'never'; // seconds, or null when forever
```

## Notes

- `Ttl::seconds()` requires a value **greater than zero** — there is no "zero means
  delete" rule; delete explicitly with `delete()`.
- Forever is stored as "no expiry", not a huge integer, so it doesn't depend on
  `PHP_INT_MAX`.

See [TTL & Clock](../api/time-builder.md) for the full reference.
