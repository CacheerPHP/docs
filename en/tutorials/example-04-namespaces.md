# Scopes (namespaces)

Scopes are isolated keyspaces. They replace v5's positional string namespaces.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

// The same key name is independent per scope.
$cache->scope('app')->set('config', $appConfig);
$cache->scope('api')->set('config', $apiConfig);

$cache->scope('app')->get('config'); // $appConfig
$cache->scope('api')->get('config'); // $apiConfig
$cache->get('config');               // null — root is a different keyspace

// Clear just one scope.
$cache->scope('api')->clear();
```

Scopes nest:

```php
$tenant = $cache->scope('tenant:42');
$tenant->scope('reports')->set('daily', $rows);
$tenant->clear(); // clears the tenant and everything under it
```

`scope()` returns another `Cacheer` — same type, same whole API — so a scoped
cache can never silently lack `remember()`, `flexible()`, batch reads,
capabilities, `withPolicy()`, or `formatted()`. `in()` is an alias if it reads
better:

```php
$cache->in('reports')->remember('daily', '10 minutes', fn () => build());
$cache->in('reports')->increment('runs');       // scoped counter
$cache->in('reports')->boundScope();            // "reports"
```

See the [Scopes guide](../guides/scopes.md).
