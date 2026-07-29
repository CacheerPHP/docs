# PSR-16 adapter

Wrap a `Cache` in `Psr16Cache` to hand any interoperable library a standard
`Psr\SimpleCache\CacheInterface`.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Psr\Psr16Cache;

$psr16 = new Psr16Cache(Cache::file(__DIR__ . '/cache'));

$psr16->set('token', 'abc123', 1800);   // ttl: int seconds, DateInterval, or null
$psr16->get('token');                    // 'abc123'
$psr16->get('missing', 'default');       // 'default'
$psr16->has('token');                    // true
$psr16->delete('token');

// Multiples
$psr16->setMultiple(['a' => 1, 'b' => 2], 3600);
$psr16->getMultiple(['a', 'b', 'c'], 'default');
$psr16->deleteMultiple(['a', 'b']);
$psr16->clear();
```

Spec behavior honored: reserved key characters (`{}()/\@:`) throw; a `null` TTL
means forever while `<= 0` deletes; a cached `null` is a hit distinct from the
default. For the pool/item model, see [PSR-6](../api/psr16-adapter.md#psr-6--psr6pool).
