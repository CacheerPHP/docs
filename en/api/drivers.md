# API Reference — Drivers

`setDriver()` returns a `CacheDriver` instance that lets you select the cache backend. CacheerPHP ships with four drivers.

> **Note:** All methods can be called statically: `Cacheer::setDriver()->useFileDriver();`

---

## Available Drivers

| Method | Backend | Best for | Requires |
|--------|---------|----------|----------|
| `useFileDriver()` | Local filesystem | General-purpose caching | `cacheDir` option |
| `useDatabaseDriver()` | MySQL / PostgreSQL / SQLite | Shared cache across servers | PDO + `.env` config |
| `useRedisDriver()` | Redis server | High-throughput / distributed | `predis/predis` + Redis |
| `useArrayDriver()` | In-memory PHP array | Testing / short-lived scripts | Nothing |
| `useDefaultDriver()` | File (auto-creates dir) | Zero-config quickstart | — |

---

## Usage

### Instance

```php
$cache = new Cacheer();
$cache->setDriver()->useFileDriver();
```

### Static

```php
Cacheer::setDriver()->useArrayDriver();
```

### With OptionBuilder

```php
use Silviooosilva\CacheerPhp\Config\Option\Builder\OptionBuilder;

$options = OptionBuilder::forRedis()
    ->setNamespace('app:')
    ->expirationTime('2 hours')
    ->build();

$cache = new Cacheer($options);
$cache->setDriver()->useRedisDriver();
```

---

## Inspecting the Active Driver *(v5.0.0)*

```php
// Get the current driver instance
$driver = $cache->getCacheStore();
echo get_class($driver);
// Silviooosilva\CacheerPhp\CacheStore\ArrayCacheStore

// Switch the driver at runtime
$cache->setCacheStore(new ArrayCacheStore($logPath));

// Check via stats()
$info = $cache->stats();
echo $info['driver'];  // full class name of the active driver
```

---

## Driver Details

### File Driver

Stores each cache item as a serialized file on disk. In v5.0.0, each file contains a JSON envelope with `{data, expires_at, ttl}` for per-item TTL support.

```php
$cache = new Cacheer(['cacheDir' => __DIR__ . '/cache']);
// File driver is the default
```

### Database Driver

Uses a PDO connection to store cache items in a database table. Configure connection details via `.env` or `setConfig()->setDatabaseConnection()`.

```php
$cache = new Cacheer();
$cache->setConfig()->setDatabaseConnection('sqlite');
$cache->setDriver()->useDatabaseDriver();
```

### Redis Driver

Connects to a Redis server via `predis/predis`. Configure host, port, and password in `.env`.

```php
$cache = new Cacheer();
$cache->setDriver()->useRedisDriver();
```

In v5.0.0, storing with `PHP_INT_MAX` TTL (forever) uses `SET` instead of `SETEX` to avoid Redis rejecting extremely large expiry values.

### Array Driver

Stores everything in a PHP array — data is lost when the process ends. Ideal for unit tests and CLI scripts.

```php
$cache = new Cacheer();
$cache->setDriver()->useArrayDriver();
```
