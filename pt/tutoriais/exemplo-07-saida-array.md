# Operações em lote

Leia, grave e apague muitas chaves de uma vez. Em stores que implementam `BatchStore`
(as quatro nativas), essas operações usam uma operação multi-chave nativa ou uma
transação em vez de um loop.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

// Grave várias sob um TTL (as chaves devem ser strings).
$cache->setMany([
    'user:1' => $u1,
    'user:2' => $u2,
    'user:3' => $u3,
], ttl: '10 minutes');

// Leia várias — misses retornam o padrão.
$users = $cache->many(['user:1', 'user:2', 'user:99'], default: null);
// ['user:1' => $u1, 'user:2' => $u2, 'user:99' => null]

// Apague várias — true só se removeu todas.
$cache->deleteMany(['user:1', 'user:2']);
```

Operações em lote respeitam o escopo atual, então
`$cache->scope('reports')->many([...])` lê dentro de `reports`.
