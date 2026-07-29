# Limpeza e flush

Três níveis de remoção: uma chave, um escopo ou a store inteira.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;

$cache = Cache::file(__DIR__ . '/cache');

// Uma chave.
$cache->delete('user:123');

// Várias chaves de uma vez.
$cache->deleteMany(['user:1', 'user:2', 'user:3']);

// Apenas um escopo.
$cache->scope('reports')->clear();

// O keyspace inteiro da store.
$cache->clear();
```

- `delete()` retorna `true` quando removeu algo.
- `clear()` em um cache **com escopo** remove apenas aquele escopo; no cache raiz
  esvazia a store inteira. Ele só toca no keyspace do próprio CacheerPHP.
- Entradas expiradas são removidas preguiçosamente na leitura, e em massa com
  [`prune()`](./exemplo-09-limpeza-automatica.md).

Veja [Escopos](./exemplo-04-namespaces.md).
