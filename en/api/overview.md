# Architecture overview

CacheerPHP 6 is built from small, composable pieces. You interact with a `Cache`
kernel; it delegates to a `Store`; optional decorators add tiering, resilience,
and instrumentation; and a storage pipeline turns values into a safe, versioned
envelope.

```text
Public API
  Cache / ScopedCache / PolicyCache
          |
Core value objects
  Key / Scope / Ttl / CacheEntry / Clock
          |
Decorators (optional, composable)
  TieredStore / ResilientStore / InstrumentedStore
          |
Store contract (+ optional capabilities)
  Store: get / set / delete / clear
          |
Built-in stores
  ArrayStore / FileStore / DatabaseStore / RedisStore
          |
Storage pipeline
  serialize -> compress -> authenticated-encrypt -> versioned Envelope
```

## Namespaces

| Namespace | Contents |
|---|---|
| `Kernel\` | `Cache`, `ScopedCache`, `PolicyCache`, and the value objects `Key`, `Scope`, `Ttl`, `CacheEntry` |
| `Contracts\` | `Store` and the capability interfaces (`BatchStore`, `TaggableStore`, `LockingStore`, `AtomicStore`, `TouchStore`, `PrunableStore`, `InspectableStore`, `FlushableScopeStore`), plus `Clock`, `Lock`, `DeferredExecutor`, `EventDispatcher`, `RedisConnection` |
| `Stores\` | `ArrayStore`, `FileStore`, `DatabaseStore`, `RedisStore`, and the decorators `TieredStore`, `ResilientStore`, `InstrumentedStore` |
| `Storage\` | `Envelope`, `EnvelopeCodec`, serializers, compression, encryption, key encoding, and the v5 reader |
| `Config\` | `PipelineConfig`, `CachePolicy` |
| `Support\` | `SystemClock`, `CircuitBreaker`, deferred executors |
| `Observability\` | `CacheEvent`, `CacheEventType`, `EventBus`, `MetricsCollector`, PSR-3/PSR-14 bridges |
| `Psr\` | `Psr16Cache`, `Psr6Pool`, `Psr6Item` |
| `Console\` | the `cacheer` operations CLI |
| `Compat\` | `LegacyCacheer` — the v5 bridge |

## The minimal Store contract

A store only has to implement four methods:

```php
interface Store
{
    public function get(Key $key): CacheEntry;
    public function set(Key $key, mixed $value, Ttl $ttl): void;
    public function delete(Key $key): bool;
    public function clear(): void;
}
```

Everything else — batching, tags, locks, atomic counters, touch, prune, inspect,
scoped flush — is an **optional capability** the store declares by implementing an
interface. See [Stores & capabilities](./drivers.md).

## Reading vs. inspecting

- `Cache::get($key, $default)` returns the value or your default. Simple.
- `Cache::entry($key)` returns a [`CacheEntry`](./option-builder.md#cacheentry)
  so you can distinguish a stored `null` from a miss and read timestamps and
  remaining TTL.

## Where configuration lives

There is no ambient configuration. A `Cache` is exactly what you construct:
a store (built from a [`PipelineConfig`](./config.md) when it persists), an
optional `Clock`, a deferred executor, and an event dispatcher. The library never
reads `.env`, changes the timezone, or creates a schema on its own.
