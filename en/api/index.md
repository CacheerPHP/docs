# API Reference

Complete reference for all CacheerPHP v5.0.0 public classes and methods.

## Core

| Page | Description |
|------|-------------|
| [Overview](./overview.md) | Architecture summary and class map |
| [Cache Functions](./cache-functions.md) | `getCache`, `putCache`, `has`, `remember`, `add`, `tag`, `stats`, and more |
| [Configuration: `setConfig()`](./config.md) | Timezone, database connection, logger path |
| [Drivers: `setDriver()`](./drivers.md) | File, Database, Redis, and Array backends |

## Configuration Helpers

| Page | Description |
|------|-------------|
| [OptionBuilder](./option-builder.md) | Fluent builder for File, Redis, and Database options |
| [TimeBuilder](./time-builder.md) | Chainable time interval helpers |

## Features

| Page | Description |
|------|-------------|
| [Compression & Encryption](./compression-encryption.md) | gzip compression and AES-256-CBC encryption |
| [PSR-16 Adapter](./psr16-adapter.md) | `Psr16CacheAdapter` — standard SimpleCache interface *(new in v5)* |

## Notes

- `expirationTime` acts as a default TTL when you omit TTL in `putCache()` (or pass the implicit `3600`). Explicit TTL values other than `3600` override the default.
- `flushAfter` enables an auto-flush check on store initialization; if the interval has elapsed the store will call `flushCache()`.
- All TTL parameters now accept `int`, `string`, `\DateInterval`, or `null` *(new in v5)*.
