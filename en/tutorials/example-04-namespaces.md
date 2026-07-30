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

A `ScopedCacheer` has the same API as `Cacheer` — `set`, `get`, `remember`,
`flexible`, batch, and `scope()` again. See the [Scopes guide](../guides/scopes.md).
