# Stores & capabilities

A **store** is where entries live. v6 keeps the required contract tiny and moves
everything optional into **capability interfaces** a store declares by
implementing them. The kernel checks for a capability and throws
`UnsupportedCapabilityException` if it is missing — it never fakes a guarantee.

## The `Store` contract

```php
namespace Silviooosilva\CacheerPhp\Contracts;

interface Store
{
    public function get(Key $key): CacheEntry;             // hit or miss
    public function set(Key $key, mixed $value, Ttl $ttl): void;
    public function delete(Key $key): bool;               // true if removed
    public function clear(): void;                        // this store's keyspace only
}
```

## Capability interfaces

| Interface | Methods | Adds |
|---|---|---|
| `BatchStore` | `getMany`, `setMany`, `deleteMany` | Native multi-key operations |
| `TouchStore` | `touch(Key, Ttl): bool` | Extend an entry's TTL in place |
| `PrunableStore` | `prune(): int` | Remove expired entries; returns the count |
| `InspectableStore` | `entries(?Scope): iterable` | Iterate live entries / metadata |
| `FlushableScopeStore` | `clearScope(Scope): void` | Clear a single scope |
| `TaggableStore` | `tag(Key, ...string)`, `clearTag(string): int` | Group + invalidate by tag |
| `AtomicStore` | `increment(Key, int $amount = 1, ?int $initial = null, ?Ttl = null): int`, `compareAndSwap(Key, mixed $expected, mixed $value, ?Ttl = null): bool` | Atomic counters and CAS |
| `LockingStore` | `lock(string $name, Ttl): Lock` | Named locks — see [Locks](./locks.md) |

## Built-in stores

All four built-in stores implement **every** capability above. What differs is the
*guarantee* behind a capability, especially atomicity and locking.

| | `ArrayStore` | `FileStore` | `DatabaseStore` | `RedisStore` |
|---|:--:|:--:|:--:|:--:|
| Persistence | In-process | Disk | PDO (SQLite/MySQL/PgSQL) | Redis server |
| Dependencies | none | none | `ext-pdo` | `predis` or `ext-redis` |
| Atomic scope | one request | one host (file lock) | row lock / txn | server-side |
| Best for | tests, short CLI | single host | shared state | high throughput |

```php
use Silviooosilva\CacheerPhp\Stores\{ArrayStore, FileStore, DatabaseStore, RedisStore};

$store = new ArrayStore($clock);
$store = new FileStore('/var/cache/app', $codec, clock: $clock);
$store = new DatabaseStore($pdo, 'cacheer_store', $codec, clock: $clock);
$store = new RedisStore($connection, 'cacheer', $codec, clock: $clock);
```

The `$codec` comes from a [`PipelineConfig`](./config.md); omit it for the safe
default (PHP serialization, no compression or encryption). Prefer the
[named constructors](./cache-functions.md#named-constructors) unless you need the
raw store.

### Redis connections

`RedisStore` takes an injected `RedisConnection` — it never creates a global
client:

```php
use Silviooosilva\CacheerPhp\Stores\Support\{PredisConnection, PhpRedisConnection};

$connection = new PredisConnection($predisClient);    // predis/predis
$connection = new PhpRedisConnection($redis);         // ext-redis \Redis
```

### Database schema

`DatabaseStore` never creates its schema implicitly. Migrate once, explicitly:

```php
use Silviooosilva\CacheerPhp\Stores\Support\DatabaseStoreSchema;

DatabaseStoreSchema::migrate($pdo, 'cacheer_store');  // idempotent
DatabaseStoreSchema::drop($pdo, 'cacheer_store');     // rollback
```

Or preview the DDL with `cacheer migrate --dry-run` (see [CLI](../guides/cli.md)).

## Decorators

Decorators wrap any store and forward every capability the wrapped store(s)
provide, so composition never loses a feature.

- **[`TieredStore`](../guides/tiered-caching.md)** — a fast local L1 in front of a
  shared L2, with promotion and generation-based coherence.
- **[`ResilientStore`](../guides/resilient-store.md)** — primary with a fallback,
  guarded by a circuit breaker.
- **[`InstrumentedStore`](../guides/observability.md)** — times every operation
  and emits typed events; values are never captured unless you opt in.

```php
$cache = Cache::tiered(new ArrayStore($clock), $redisStore);
$cache = Cache::resilient($primary, $fallback);
$cache = Cache::instrumented($store, $events);
```

## Writing your own

Implement `Store`, add the capabilities you can honor, and prove it with the
shared conformance suite. See [Custom stores](../guides/custom-stores.md).
