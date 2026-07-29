# PSR-16 & PSR-6 adapters

CacheerPHP 6 ships adapters for both cache PSRs over the same `Cache` kernel, so
you can hand a standards-compliant cache to any interoperable library.

## PSR-16 — `Psr16Cache`

`Silviooosilva\CacheerPhp\Psr\Psr16Cache` implements
`Psr\SimpleCache\CacheInterface`.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Psr\Psr16Cache;

$psr16 = new Psr16Cache(Cache::file('/var/cache/app'));

$psr16->set('token', 'abc123', 1800);        // ttl: int seconds, DateInterval, or null (forever)
$psr16->get('token');                         // 'abc123'
$psr16->get('missing', 'default');            // 'default'
$psr16->has('token');                         // true
$psr16->delete('token');

$psr16->setMultiple(['a' => 1, 'b' => 2], 3600);
$psr16->getMultiple(['a', 'b', 'c'], 'default');
$psr16->deleteMultiple(['a', 'b']);
$psr16->clear();
```

Spec details honored:

- Reserved key characters (`{}()/\@:`) throw `CacheInvalidArgumentException`
  (which implements the PSR-16 *and* PSR-6 `InvalidArgumentException`).
- A `null` or non-positive integer TTL: `null` means forever; `<= 0` deletes the
  key (as the spec requires).
- A cached `null` is returned as a hit, distinct from the default on a miss.

## PSR-6 — `Psr6Pool`

`Silviooosilva\CacheerPhp\Psr\Psr6Pool` implements
`Psr\Cache\CacheItemPoolInterface`; items are `Psr6Item`. The pool takes a
`Cache` and a `Clock`.

```php
use Silviooosilva\CacheerPhp\Psr\Psr6Pool;
use Silviooosilva\CacheerPhp\Support\SystemClock;

$pool = new Psr6Pool(Cache::file('/var/cache/app'), new SystemClock());

$item = $pool->getItem('user:42');
if (! $item->isHit()) {
    $item->set($user)->expiresAfter(600); // seconds or a DateInterval
    $pool->save($item);
}
$user = $pool->getItem('user:42')->get();

// Deferred saves are flushed on commit().
$pool->saveDeferred($pool->getItem('a')->set(1));
$pool->commit();
```

`Psr6Item::expiresAfter()` is clock-agnostic (relative), while
`expiresAt(DateTimeInterface)` pins an absolute expiry; the pool resolves both
against its injected clock, so PSR-6 expiry is deterministic under a `FakeClock`.

## Which one?

- Use **PSR-16** for straightforward key/value caching and the widest library
  support.
- Use **PSR-6** when a library requires the pool/item model or deferred commits.
- Use the native [`Cache`](./cache-functions.md) API for scopes, `remember()`,
  `flexible()`, policies, tiering, and resilience — features the PSRs don't cover.
