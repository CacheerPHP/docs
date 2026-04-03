# Getting Started

CacheerPHP is a lightweight PHP caching library with four storage backends (File, Database, Redis, Array), flexible TTL handling, optional compression and AES-256-CBC encryption, PSR-16 compliance, and a fluent OptionBuilder.

## Requirements

| Requirement | Details |
|-------------|---------|
| **PHP** | 8.2 or newer |
| **Extensions** | `ext-openssl`, `ext-zlib`, `ext-pdo` |
| **Optional** | PDO drivers for MySQL, PostgreSQL, or SQLite |
| **Optional** | Redis server + `predis/predis` for the Redis backend |

## Installation

```sh
composer require silviooosilva/cacheer-php
```

This will pull in the required PSR packages automatically (`psr/simple-cache` ^3.0, `psr/log` ^3.0).

## Quick Start — Instance Usage

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

// 1. Create a Cacheer instance with the File driver
$cache = new Cacheer(['cacheDir' => __DIR__ . '/cache']);

// 2. Store a value (TTL defaults to 3600 seconds)
$cache->putCache('user:123', ['name' => 'John Doe', 'email' => 'john@example.com']);

// 3. Retrieve it
if ($cache->isSuccess()) {
    $user = $cache->getCache('user:123');
    print_r($user);
} else {
    echo $cache->getMessage();
}

// 4. Check existence
$cache->has('user:123');  // sets isSuccess() to true/false

// 5. Remove a single key or flush everything
$cache->clearCache('user:123');
$cache->flushCache();
```

## Quick Start — Static Facade

Every method can also be called statically. CacheerPHP manages a shared singleton behind the scenes.

```php
<?php
use Silviooosilva\CacheerPhp\Cacheer;

Cacheer::setConfig()->setTimeZone('UTC');
Cacheer::setDriver()->useArrayDriver();

Cacheer::putCache('greeting', 'Hello, world!');
echo Cacheer::getCache('greeting');  // Hello, world!
```

## Choosing a Driver

```php
$cache = new Cacheer();

// File (default) — stores serialized files on disk
$cache->setDriver()->useFileDriver();

// Database — MySQL, PostgreSQL, or SQLite via PDO
$cache->setDriver()->useDatabaseDriver();

// Redis — requires a running Redis server
$cache->setDriver()->useRedisDriver();

// Array — in-memory, ideal for testing
$cache->setDriver()->useArrayDriver();
```

## TTL (Time-To-Live)

v5.0.0 accepts four TTL formats anywhere a TTL is expected:

```php
// Integer — seconds
$cache->putCache('key', $data, ttl: 3600);

// String — human-readable
$cache->putCache('key', $data, ttl: '2 hours');

// DateInterval — PHP native
$cache->putCache('key', $data, ttl: new \DateInterval('PT30M'));

// null — store forever (PHP_INT_MAX)
$cache->putCache('key', $data, ttl: null);
```

## PSR-16 SimpleCache

Wrap any Cacheer instance in the PSR-16 adapter for framework interoperability:

```php
use Silviooosilva\CacheerPhp\Psr\Psr16CacheAdapter;

$psr = new Psr16CacheAdapter($cache);

$psr->set('token', 'abc123', 1800);
$psr->get('token');           // 'abc123'
$psr->has('token');           // true
$psr->delete('token');
```

See the full [PSR-16 Adapter reference](../api/psr16-adapter.md).

## Next Steps

- [API Reference](../api/index.md) — complete method documentation
- [Tutorials](../tutorials/index.md) — step-by-step examples
- [Configuration Guide](../guides/configuration.md) — environment variables
- [Upgrading to v5](../updating/v5-migration.md) — migration from v4.x
