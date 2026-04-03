# API Reference — Overview

## Main Class

```
Silviooosilva\CacheerPhp\Cacheer
```

The facade class for all caching operations. Supports both instance usage (`$cache->method()`) and static calls (`Cacheer::method()`).

---

## Architecture at a Glance

```
Cacheer (facade)
├── CacheMutator      — write operations (put, clear, flush, add, increment, tag)
├── CacheRetriever    — read operations (get, getAll, getMany, remember)
├── CacheConfig       — timezone, driver, logger path
├── CacheDriver       — backend selection
│   ├── FileCacheStore
│   ├── DatabaseCacheStore
│   ├── RedisCacheStore
│   └── ArrayCacheStore
├── Psr16CacheAdapter — PSR-16 CacheInterface wrapper (v5.0.0)
└── CacheLogger       — PSR-3 AbstractLogger (v5.0.0)
```

---

## API Sections

### 1. Configuration

`setConfig()` returns a `CacheConfig` instance for timezone, database connection, and logger path.

[Full reference](./config.md)

### 2. Drivers

`setDriver()` returns a `CacheDriver` instance for selecting the backend.

| Method | Backend | Requires |
|--------|---------|----------|
| `useFileDriver()` | Local filesystem | `cacheDir` option |
| `useDatabaseDriver()` | MySQL / PostgreSQL / SQLite | PDO + configured `.env` |
| `useRedisDriver()` | Redis server | `predis/predis` |
| `useArrayDriver()` | In-memory PHP array | Nothing (ideal for tests) |

[Full reference](./drivers.md)

### 3. OptionBuilder

Fluent builders that eliminate typos and centralize configuration:

- `OptionBuilder::forFile()` — `dir()`, `expirationTime()`, `flushAfter()`
- `OptionBuilder::forRedis()` — `setNamespace()`, `expirationTime()`, `flushAfter()`
- `OptionBuilder::forDatabase()` — `table()`, `expirationTime()`, `flushAfter()`

[Full reference](./option-builder.md) | [TimeBuilder](./time-builder.md)

### 4. Cache Functions

All read/write operations, tagging, computed values, and diagnostics.

[Full reference](./cache-functions.md)

### 5. Compression & Encryption

- `useCompression()` — gzip via `gzcompress`
- `useEncryption(string $key)` — AES-256-CBC with random IV per write (v5.0.0)

[Full reference](./compression-encryption.md)

### 6. PSR-16 Adapter *(new in v5.0.0)*

`Psr16CacheAdapter` wraps any `Cacheer` instance into a standard `\Psr\SimpleCache\CacheInterface`.

[Full reference](./psr16-adapter.md)

### 7. Diagnostics *(new in v5.0.0)*

| Method | Description |
|--------|-------------|
| `stats()` | Returns driver class, compression flag, and encryption flag |
| `getCacheStore()` | Returns the active `CacheerInterface` driver |
| `setCacheStore($driver)` | Swaps the driver at runtime |
| `getOption($key, $default)` | Returns a single option value, or `$default` if not set |
| `getOptions()` | Returns the current options array |
| `setOption($key, $value)` | Sets a single option |
| `setOptions($array)` | Replaces all options |
| `Cacheer::resetInstance()` | Clears the static singleton |
| `Cacheer::setInstance($obj)` | Injects a custom singleton |
