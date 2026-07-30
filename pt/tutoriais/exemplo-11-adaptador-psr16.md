# Adaptador PSR-16

Embrulhe um `Cacheer` em `Psr16Cache` para entregar a qualquer biblioteca interoperável
um `Psr\SimpleCache\CacheInterface` padrão.

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Psr\Psr16Cache;

$psr16 = new Psr16Cache(Cacheer::file(__DIR__ . '/cache'));

$psr16->set('token', 'abc123', 1800);   // ttl: int segundos, DateInterval ou null
$psr16->get('token');                    // 'abc123'
$psr16->get('missing', 'default');       // 'default'
$psr16->has('token');                    // true
$psr16->delete('token');

// Múltiplos
$psr16->setMultiple(['a' => 1, 'b' => 2], 3600);
$psr16->getMultiple(['a', 'b', 'c'], 'default');
$psr16->deleteMultiple(['a', 'b']);
$psr16->clear();
```

Comportamento de spec honrado: caracteres reservados na chave (`{}()/\@:`) lançam; TTL
`null` significa para sempre enquanto `<= 0` apaga; um `null` cacheado é um hit distinto
do padrão. Para o modelo pool/item, veja [PSR-6](../api/psr16-adapter.md#psr-6--psr6pool).
