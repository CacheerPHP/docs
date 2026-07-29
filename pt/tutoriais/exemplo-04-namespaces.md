# Escopos (namespaces)

Escopos são keyspaces isolados. Eles substituem os namespaces posicionais por string
da v5.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;

$cache = Cache::file(__DIR__ . '/cache');

// O mesmo nome de chave é independente por escopo.
$cache->scope('app')->set('config', $appConfig);
$cache->scope('api')->set('config', $apiConfig);

$cache->scope('app')->get('config'); // $appConfig
$cache->scope('api')->get('config'); // $apiConfig
$cache->get('config');               // null — a raiz é outro keyspace

// Limpar apenas um escopo.
$cache->scope('api')->clear();
```

Escopos aninham:

```php
$tenant = $cache->scope('tenant:42');
$tenant->scope('reports')->set('daily', $rows);
$tenant->clear(); // limpa o tenant e tudo abaixo dele
```

Um `ScopedCache` tem a mesma API que `Cache` — `set`, `get`, `remember`, `flexible`,
lotes e `scope()` de novo. Veja o [guia de Escopos](../guias/escopos.md).
