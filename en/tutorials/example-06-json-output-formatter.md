# The CacheEntry object

> v5 had output formatters (`toJson`, `toArray`, `toString`). v6 stores values
> losslessly instead — you get back exactly what you put in. When you need
> metadata about an entry, use `entry()`.

`get()` returns the value; `entry()` returns a `CacheEntry` with hit/miss,
timestamps, and remaining TTL.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Support\SystemClock;

$cache = Cache::inMemory();
$cache->set('user:42', ['id' => 42, 'name' => 'Ada'], ttl: '10 minutes');

$entry = $cache->entry('user:42');

$entry->isHit();                          // true
$entry->value();                          // ['id' => 42, 'name' => 'Ada']
$entry->createdAt();                      // unix timestamp of the write
$entry->expiresAt();                      // unix timestamp, or null if forever
$entry->remainingTtl(new SystemClock());  // seconds left, or null if forever
```

Formatting is your application's job now:

```php
$json = json_encode($cache->get('user:42'));
```

`entry()` is also how you distinguish a **stored `null`** from a **miss** — see
[Falsy and null values](./example-13-falsy-values.md).
