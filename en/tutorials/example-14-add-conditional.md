# Example 14 — Conditional Store with add()

*Fixed in v5.0.0*

The `add()` method stores a value **only if the key does not already exist**. It returns `true` when the value was stored and `false` when the key was already present.

> **Breaking change:** In v4.x the return values were inverted. See the [migration guide](../updating/v5-migration.md) for details.

## Basic Usage

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useArrayDriver();

// First call — key does not exist, value is stored
$stored = $cache->add('lock', getmypid(), ttl: 30);
var_dump($stored); // true

// Second call — key already exists, nothing happens
$stored = $cache->add('lock', 99999, ttl: 30);
var_dump($stored); // false

// Original value is preserved
echo $cache->getCache('lock'); // prints the first PID
```

## Use Case: Simple Lock

```php
function acquireLock(Cacheer $cache, string $resource, int $ttlSeconds = 10): bool
{
    return $cache->add("lock:{$resource}", getmypid(), ttl: $ttlSeconds);
}

function releaseLock(Cacheer $cache, string $resource): void
{
    $cache->clearCache("lock:{$resource}");
}

if (acquireLock($cache, 'report-generation')) {
    try {
        // Only one process runs this at a time
        generateReport();
    } finally {
        releaseLock($cache, 'report-generation');
    }
} else {
    echo "Another process is already generating the report.";
}
```

## Use Case: Default Settings

```php
// Initialize defaults without overwriting user customizations
$cache->add('settings:theme', 'light');
$cache->add('settings:language', 'en');
$cache->add('settings:per_page', 25);

// If the user already chose "dark", the line above is a no-op
echo $cache->getCache('settings:theme'); // "dark" (user's choice preserved)
```
