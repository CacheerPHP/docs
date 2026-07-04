# CacheerPHP Documentation

**Current version: 5.2.0** | PHP 8.2+ | [Changelog](./updating/v5-migration.md)

CacheerPHP is a lightweight, driver-agnostic caching library for PHP. It ships with four built-in backends (File, Database, Redis, Array), a fluent configuration API, optional compression and AES-256-CBC encryption, PSR-16 compliance, and PSR-3 logging — all behind a single, consistent interface.

---

## Sections

| Section | Description |
|---------|-------------|
| [Getting Started](./getting-started/index.md) | Installation, requirements, and a quick-start example |
| [Guides](./guides/configuration.md) | Environment variables, database and Redis settings |
| [API Reference](./api/index.md) | Complete method reference for every public class |
| [Tutorials](./tutorials/index.md) | Step-by-step examples for common use cases |
| [Upgrading to v5](./updating/v5-migration.md) | Breaking changes, new features, and migration steps |
| [Contributing](./contributing/index.md) | How to set up, test, and submit pull requests |
| [Updating](./updating/index.md) | General upgrade procedures |

## What's New in v5.2.0

- **Distributed locks** — `Cacheer::lock('name')` returns a driver-backed mutex (`acquire`, `release`, `block`, `get`) so only one process runs a critical section at a time. Native on Redis (`SET NX`), Database (locks table), and File (`flock`); see [Distributed Locks](./api/locks.md).
- **Atomic `increment()` / `decrement()`** — counter updates are serialised on a per-key lock, so concurrent increments no longer lose updates.
- **Stampede-safe `remember()`** — a concurrent miss now runs the callback once instead of once per request (no dogpile).
- **`flexible()` — stale-while-revalidate** — serve fresh values directly, serve stale ones while a single worker refreshes, recompute once expired. See [Cache Functions → flexible()](./api/cache-functions.md#flexible--stale-while-revalidate).
- **Database falsy-value fix** — storing over an existing `0` / `false` / `''` value on the Database driver now updates correctly instead of failing.

## What's New in v5.0.0

- **PSR-16 adapter** — use CacheerPHP anywhere a `\Psr\SimpleCache\CacheInterface` is expected
- **PSR-3 logger** — `CacheLogger` now extends `\Psr\Log\AbstractLogger`
- **DateInterval TTL** — pass `int`, `string`, `\DateInterval`, or `null` to any TTL parameter
- **Random IV encryption** — every `putCache()` generates a fresh IV (AES-256-CBC)
- **Corrected `add()` semantics** — returns `true` when stored, `false` when the key already exists
- **Falsy value caching** — `0`, `false`, `''`, `[]` are now valid cache hits
- **Per-item TTL** — FileCacheStore stores expiry in the cache envelope, not `filemtime`
- **Encapsulation** — `$cacheStore` and `$options` are now private; use `getCacheStore()`, `getOptions()`, `setOption()`
- **Instance management** — `stats()`, `resetInstance()`, `setInstance()` for diagnostics and testing
- **PHP 8.2 minimum** — takes advantage of `readonly` properties and modern PHP features

> **Upgrading from v4?** See the [v5.0.0 Migration Guide](./updating/v5-migration.md) for breaking changes and step-by-step instructions.

---

Contributions are welcome. See the root README for structure and guidelines.
