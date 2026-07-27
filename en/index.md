# CacheerPHP Documentation

**Current version: 6.0** | PHP 8.3+ | [Migration guide](./updating/index.md)

CacheerPHP 6 is an **instance-first** cache: a small `Cache` kernel over a
minimal four-method `Store` contract, with everything else — batching, tags,
locks, atomic counters, tiering, resilience, encryption — an **optional
capability** you opt into. There is no global state and no autoload-time side
effect. v5 code keeps working through the `LegacyCacheer` bridge.

---

## What's New in v6.0

- **Instance-first kernel.** Explicit `Cache` and immutable `ScopedCache` over
  typed `Key`, `Scope`, `Ttl`, and `CacheEntry`; time is an injected `Clock`.
- **Tiny core, honest capabilities.** A store implements four methods; extra
  behavior is declared by interface and checked at runtime, so a backend never
  fakes a guarantee it can't make.
- **Composable decorators.** Tiered (L1/L2), resilient (circuit-breaker
  fallback), and instrumented (typed events + metrics) wrap any store.
- **Stampede protection.** Single-flight `remember()` and stale-while-revalidate
  `flexible()` are built in.
- **Authenticated storage.** serialize → optional gzip → optional AES-256-GCM
  into a versioned, tamper-evident envelope, with key rotation.
- **Standards & tooling.** PSR-16 and PSR-6 adapters, PSR-3 logging, a PSR-14
  bridge, and a `cacheer` operations CLI.
- **Migration.** `LegacyCacheer` bridge, an optional Rector rename set, and
  rewrite-on-read for v5 payloads. See the [migration guide](./updating/index.md).

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
