# Escopos (namespaces)

Escopos são keyspaces isolados. Eles substituem os namespaces posicionais por string
da v5.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

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

`scope()` retorna outro `Cacheer` — mesmo tipo, mesma API inteira — então um cache
com escopo nunca perde `remember()`, `flexible()`, leituras em lote, capacidades,
`withPolicy()` ou `formatted()` silenciosamente. `in()` é um alias, se ler melhor:

```php
$cache->in('reports')->remember('daily', '10 minutes', fn () => build());
$cache->in('reports')->increment('runs');       // contador com escopo
$cache->in('reports')->boundScope();            // "reports"
```

Veja o [guia de Escopos](../guias/escopos.md).
