# Falsy and null values

A cached `null`, `false`, `0`, `''`, or `[]` is a **hit**, returned exactly as
stored. Only `entry()` can distinguish a stored `null` from a miss.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;

$cache = Cache::inMemory();

$cache->set('flag', false);
$cache->set('count', 0);
$cache->set('nothing', null);

$cache->get('flag');   // false (a hit, not a miss)
$cache->get('count');  // 0
$cache->get('nothing'); // null
```

Because `get('nothing')` and `get('absent')` both return `null`, use `entry()`
when the difference matters:

```php
$cache->entry('nothing')->isHit(); // true  — a stored null
$cache->entry('absent')->isHit();  // false — a real miss
```

Or pass a sentinel default:

```php
$missing = new stdClass();
if ($cache->get('nothing', $missing) === $missing) {
    // truly absent
}
```

This is why `remember()` won't recompute a legitimately empty result on every
call — a stored `null` counts as a hit.
