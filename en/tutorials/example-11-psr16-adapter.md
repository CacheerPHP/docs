# Example 11 — PSR-16 SimpleCache Adapter

*New in v5.0.0*

CacheerPHP ships with a PSR-16 adapter that lets you use it anywhere a standard `\Psr\SimpleCache\CacheInterface` is expected.

## Basic Usage

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Psr\Psr16CacheAdapter;

$cacheer = new Cacheer(['cacheDir' => __DIR__ . '/cache']);
$cache   = new Psr16CacheAdapter($cacheer);

// Store and retrieve
$cache->set('greeting', 'Hello, PSR-16!', 3600);
echo $cache->get('greeting');          // Hello, PSR-16!
echo $cache->has('greeting');          // true

// Default values for misses
echo $cache->get('missing', 'fallback');  // fallback
```

## Batch Operations

```php
$cache->setMultiple([
    'user:1' => ['name' => 'Alice'],
    'user:2' => ['name' => 'Bob'],
], 1800);

$users = $cache->getMultiple(['user:1', 'user:2', 'user:99'], 'NOT FOUND');
// user:99 => 'NOT FOUND'

$cache->deleteMultiple(['user:1', 'user:2']);
```

## Namespace Isolation

Different adapter instances can share the same Cacheer backend while keeping keys separate:

```php
$users    = new Psr16CacheAdapter($cacheer, 'users');
$sessions = new Psr16CacheAdapter($cacheer, 'sessions');

$users->set('config', 'user-value');
$sessions->set('config', 'session-value');

echo $users->get('config');     // user-value
echo $sessions->get('config');  // session-value
```

## Key Validation

PSR-16 forbids certain characters in keys. Invalid keys throw `CacheInvalidArgumentException`:

```php
use Silviooosilva\CacheerPhp\Exceptions\CacheInvalidArgumentException;

try {
    $cache->get('bad:key');
} catch (CacheInvalidArgumentException $e) {
    echo $e->getMessage();
}
```

See the full [PSR-16 Adapter API reference](../api/psr16-adapter.md).
