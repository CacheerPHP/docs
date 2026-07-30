# CacheerPHP Documentation

**Current version: 6.0** | PHP 8.3+ | [Migration guide](./updating/index.md)

CacheerPHP 6 is an **instance-first** cache: a small `Cacheer` kernel over a minimal
four-method `Store` contract, with everything else — batching, tags, locks, atomic
counters, tiering, resilience, encryption — an **optional capability** you opt
into. There is no global state and no autoload-time side effect. v5 stays on its
own `5.x` line during migration.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

$cache->set('user:42', $user, ttl: '10 minutes');
$user = $cache->get('user:42');

$user = $cache->remember('user:42', '10 minutes', fn () => $users->find(42));
```

---

## Sections

| Section | Description |
|---------|-------------|
| [Getting Started](./getting-started/index.md) | Install, quick start, stores, scopes, TTL, PSR |
| [Guides](./guides/configuration.md) | Deep dives: scopes, remember/locks, SWR, tiering, resilience, policies, encryption, observability, CLI, custom stores |
| [API Reference](./api/index.md) | Precise reference for every public class and method |
| [Tutorials](./tutorials/index.md) | Short, task-focused v6 examples |
| [Migration guide](./updating/index.md) | Upgrading from v5 |
| [Contributing](./contributing/index.md) | Setup, tests, conformance, RFCs |

## What's New in v6.0

- **Instance-first kernel.** Explicit `Cacheer` and immutable `ScopedCacheer` over
  typed `Key`, `Scope`, `Ttl`, and `CacheEntry`; time is an injected `Clock`.
- **Tiny core, honest capabilities.** A store implements four methods; extra
  behavior is declared by interface and checked at runtime, so a backend never
  fakes a guarantee it can't make.
- **Scopes** replace stringly namespaces with isolated keyspaces you can clear on
  their own.
- **Composable decorators.** [Tiered](./guides/tiered-caching.md) (L1/L2),
  [resilient](./guides/resilient-store.md) (circuit-breaker fallback), and
  [instrumented](./guides/observability.md) (typed events + metrics) wrap any store.
- **Stampede protection.** Single-flight [`remember()`](./guides/remember-and-locks.md)
  and stale-while-revalidate [`flexible()`](./guides/stale-while-revalidate.md).
- **Authenticated storage.** serialize → optional gzip → optional AES-256-GCM into
  a versioned, tamper-evident envelope, with [key rotation](./guides/encryption-and-compression.md).
- **Standards & tooling.** [PSR-16 and PSR-6](./api/psr16-adapter.md) adapters,
  PSR-3 logging, a PSR-14 bridge, and a [`cacheer` CLI](./guides/cli.md).
- **Migration.** An optional Rector rename set plus rewrite-on-read for v5
  payloads — the rename is the migration, no runtime shim. See the
  [migration guide](./updating/index.md).

## Breaking changes at a glance

- The static/global facade is gone — construct and inject a `Cacheer`. No drop-in
  v5 shim; migrate with the Rector set + mapping, or stay on `^5.2`.
- `get()` no longer takes a read-time TTL; positional namespaces become `scope()`;
  success is a return value or `entry()->isHit()`, not mutable state.
- Minimum PHP is **8.3**. Driver clients and extensions are optional.

---

Contributions are welcome. See the root README for structure and guidelines.
