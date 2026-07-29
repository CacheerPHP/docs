# Batch operations

Read, write, and delete many keys at once. On stores that implement `BatchStore`
(all four built-ins), these use a native multi-key operation or a transaction
instead of a loop.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;

$cache = Cache::file(__DIR__ . '/cache');

// Write many under one TTL (keys must be strings).
$cache->setMany([
    'user:1' => $u1,
    'user:2' => $u2,
    'user:3' => $u3,
], ttl: '10 minutes');

// Read many — misses return the default.
$users = $cache->many(['user:1', 'user:2', 'user:99'], default: null);
// ['user:1' => $u1, 'user:2' => $u2, 'user:99' => null]

// Delete many — true only if every key was removed.
$cache->deleteMany(['user:1', 'user:2']);
```

Batch operations respect the current scope, so
`$cache->scope('reports')->many([...])` reads within `reports`.
