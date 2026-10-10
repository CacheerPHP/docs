# CacheerPHP Documentation

**Documentation: 6.x stable** | PHP 8.3+ | [Migration guide](./updating/index.md)

CacheerPHP 6 is an **instance-first** cache: a small `Cacheer` kernel over a minimal
four-method `Store` contract, with everything else — batching, tags, locks, atomic
counters, tiering, resilience, encryption — an **optional capability** you opt
into. Caches are never global — you construct one and inject it — and the package
itself runs nothing at autoload time; the one process-global is the opt-in
[`Telemetry`](./guides/observability.md#the-global-telemetry-tap) tap, dormant
until a listener is registered. v5 stays on its own `5.x` line during migration.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

$cache->set('user:42', $user, ttl: '10 minutes');
$user = $cache->get('user:42');

$user = $cache->remember('user:42', '10 minutes', fn () => $users->find(42));
```

---

## Choose a task

| I want to… | Start here |
|---|---|
| Install CacheerPHP | [Getting started](./getting-started/index.md) |
| Choose storage | [Stores and capabilities](./api/drivers.md) |
| Cache expensive results | [Remember and locks](./guides/remember-and-locks.md) |
| Handle concurrent requests | [Stale-while-revalidate](./guides/stale-while-revalidate.md) |
| Monitor and debug | [Cacheer Monitor quick start](./cacheer-monitor/quick-start.md) |
| Migrate a v5 application | [Migration guide](./updating/index.md) |

For method signatures, use the [API reference](./api/index.md). For runnable examples, browse the [tutorials](./tutorials/index.md). Check [releases and version support](./updating/version-support.md) before selecting a version.

## What's New in v6.0

- **Instance-first kernel.** One explicit, immutable `Cacheer` behind a `Cache`
  interface, over typed `Key`, `Scope`, `Ttl`, and `CacheEntry`; time is an
  injected `Clock`.
- **One cache type.** Scope and policy are state on the object, so every
  combination composes: `$cache->in('billing')->withPolicy($p)->increment('hits')`.
  Type-hint `Cache` and any of them is substitutable.
- **Tiny core, honest capabilities.** A store implements four methods; extra
  behavior is declared by interface and `$cache->supports(...)` answers truthfully
  even through decorators, so a backend never fakes a guarantee it can't make.
- **Capabilities on the cache.** `increment`, `decrement`, `touch`, `tag`,
  `flushTag`, `lock`, `entries`, and `prune` are methods on the cache, with the
  scope applied — no reaching past it to the store.
- **Scopes** replace stringly namespaces with isolated keyspaces you can clear on
  their own — and they apply to counters, tags, and locks too.
- **Composable decorators.** [Tiered](./guides/tiered-caching.md) (L1/L2),
  [resilient](./guides/resilient-store.md) (circuit-breaker fallback), and
  [instrumented](./guides/observability.md) (typed events + metrics) wrap any store.
- **Stampede protection.** Single-flight [`remember()`](./guides/remember-and-locks.md)
  and stale-while-revalidate [`flexible()`](./guides/stale-while-revalidate.md).
- **Authenticated storage.** serialize → optional gzip → optional AES-256-GCM into
  a versioned, tamper-evident envelope, with [key rotation](./guides/encryption-and-compression.md).
- **Standards & tooling.** [PSR-16 and PSR-6](./api/psr16-adapter.md) adapters,
  PSR-3 logging, a PSR-14 bridge, and a [`cacheer` CLI](./guides/cli.md).
- **Migration.** An optional Rector rename set and a cold-keyspace upgrade — the
  rename is the migration, no runtime shim. See the
  [migration guide](./updating/index.md).

## Breaking changes at a glance

- The static/global facade is gone — construct and inject a `Cacheer`. No drop-in
  v5 shim; migrate with the Rector set + mapping, or stay on `^5.2`.
- `get()` no longer takes a read-time TTL; positional namespaces become `scope()`
  (or its alias `in()`); success is a return value or `entry()->isHit()`, not
  mutable state.
- Familiar v5 verbs are still here: `forever()`, `rememberForever()`, `missing()`,
  `add()`, `pull()` (v5's `getAndForget`), `increment()`/`decrement()`, `touch()`
  (v5's `renewCache`), `tag()`/`flushTag()`, `lock()`, and `stats()`.
- Minimum PHP is **8.3**. Driver clients and extensions are optional.

---

Contributions are welcome. See the root README for structure and guidelines.
