# API Reference

This page summarizes the main APIs and points to detailed topics.

- [Core configuration: `setConfig()`](./config.md)
- [Driver selection: `setDriver()`](./drivers.md)
- [Fluent options: `OptionBuilder`](./option-builder.md)
- [Time helpers: `TimeBuilder`](./time-builder.md)
- [Compression & encryption](./compression-encryption.md)
- [Cache functions (get/put/has/flush/etc.)](./cache-functions.md)

Notes
- `expirationTime` acts as a default TTL when you omit TTL in `putCache()` (or pass the implicit 3600). Explicit TTL values other than 3600 override the default.
- `flushAfter` enables an auto-flush check on store initialization; if the interval has elapsed the store will call `flushCache()`.

If you prefer a single-page overview, see [overview](./overview.md) (legacy index).
