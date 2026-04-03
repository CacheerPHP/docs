# Example 15 — Stats and Instance Management

*New in v5.0.0*

CacheerPHP v5.0.0 adds diagnostic and lifecycle methods that make it easier to inspect, test, and manage cache instances.

## stats() — Inspect the Cache State

`stats()` returns an associative array with the active driver, compression, and encryption status:

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useArrayDriver();
$cache->useCompression();

$info = $cache->stats();
print_r($info);
// [
//     'driver'      => 'Silviooosilva\CacheerPhp\CacheStore\ArrayCacheStore',
//     'compression' => true,
//     'encryption'  => false,
// ]
```

## getCacheStore() / setCacheStore() — Runtime Driver Swap

You can retrieve or replace the underlying store at runtime:

```php
use Silviooosilva\CacheerPhp\CacheStore\ArrayCacheStore;

// Get the current store
$store = $cache->getCacheStore();
echo get_class($store);

// Swap to a different store at runtime
$cache->setCacheStore(new ArrayCacheStore('/tmp/cacheer.log'));
```

## resetInstance() — Clear Static Singleton

When using CacheerPHP via static calls, the singleton persists for the lifetime of the process. Use `resetInstance()` to clear it — useful in tests or long-running workers:

```php
// Static usage creates a singleton
Cacheer::setDriver()->useArrayDriver();
Cacheer::putCache('key', 'value');

// Reset for a clean slate
Cacheer::resetInstance();

// Next static call creates a fresh instance
Cacheer::setDriver()->useArrayDriver();
var_dump(Cacheer::has('key')); // false — previous data is gone
```

## setInstance() — Inject a Custom Instance

Replace the static singleton with a pre-configured instance. Helpful for testing or dependency injection:

```php
$testCache = new Cacheer();
$testCache->setDriver()->useArrayDriver();
$testCache->putCache('fixture', 'data');

// Inject into the static singleton
Cacheer::setInstance($testCache);

// Static calls now use the injected instance
echo Cacheer::getCache('fixture'); // "data"
```

## getOptions() / setOption() / setOptions()

Inspect or modify configuration at runtime:

```php
$cache = new Cacheer(['cacheDir' => __DIR__ . '/cache']);

// Read all options
$opts = $cache->getOptions();

// Set a single option
$cache->setOption('expirationTime', '2 hours');

// Replace all options
$cache->setOptions([
    'cacheDir' => '/tmp/cache',
    'expirationTime' => '30 minutes',
]);
```
