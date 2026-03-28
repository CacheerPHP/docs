# API Reference — Configuration

`setConfig()` returns a `CacheConfig` instance with methods for timezone, database connection, logger path, and option management.

> **Tip:** Always set the driver before configuring. See [Drivers](./drivers.md).

> **Note:** All methods can be called statically: `Cacheer::setConfig()->setTimeZone('UTC');`

---

## Database Connection

```php
$cache->setConfig()->setDatabaseConnection(string $driver): void
```

Sets the PDO driver used for the Database cache store.

| Parameter | Values | Default |
|-----------|--------|---------|
| `$driver` | `'mysql'`, `'pgsql'`, `'sqlite'` | From `.env` `DB_CONNECTION` |

```php
$cache->setConfig()->setDatabaseConnection('mysql');
// or
Cacheer::setConfig()->setDatabaseConnection('sqlite');
```

Alternatively, set `DB_CONNECTION` in your `.env` file.

---

## Timezone

```php
$cache->setConfig()->setTimeZone(string $timezone): CacheConfig
```

Sets the default timezone for cache expiry calculations. Must be a valid [PHP timezone identifier](https://www.php.net/manual/en/timezones.php).

```php
$cache->setConfig()->setTimeZone('UTC');
$cache->setConfig()->setTimeZone('America/Sao_Paulo');
```

---

## Logger Path

```php
$cache->setConfig()->setLoggerPath(string $path): Cacheer
```

Defines the file path for the cache logger. The logger implements PSR-3 `\Psr\Log\AbstractLogger` (v5.0.0) and supports automatic log rotation.

```php
$cache->setConfig()->setLoggerPath('/var/log/cacheer.log');
```

---

## Options Management *(v5.0.0)*

### `setUp()` — Replace all options

```php
$cache->setUp(array $options): void
// or via setConfig():
$cache->setConfig()->setUp($options);
```

Replaces the entire options array. Commonly used with `OptionBuilder`:

```php
use Silviooosilva\CacheerPhp\Config\Option\Builder\OptionBuilder;

$options = OptionBuilder::forFile()
    ->dir(__DIR__ . '/cache')
    ->expirationTime('2 hours')
    ->build();

$cache->setUp($options);
```

### `getOption()` — Read a single option *(new in v5)*

```php
$cache->getOption(string $key, mixed $default = null): mixed
```

Returns the value of a single configuration option, or `$default` if the key is not set.

```php
$dir = $cache->getOption('cacheDir', '/tmp/cache');
```

### `getOptions()` — Read current options

```php
$cache->getOptions(): array
// or via setConfig():
$cache->setConfig()->getOptions();
```

Returns the full options array.

### `setOption()` — Set a single option *(new in v5)*

```php
$cache->setOption(string $key, mixed $value): Cacheer
```

### `setOptions()` — Replace all options *(new in v5)*

```php
$cache->setOptions(array $options): void
```

> **Note:** In v5.0.0, `$cache->options` is private. Use these accessors instead of direct property access.
