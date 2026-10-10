# Upgrading to CacheerPHP 6

CacheerPHP 6 is an instance-first rewrite. Migrating is mostly mechanical: rename
the v5 methods to the v6 names (a Rector set automates the common ones), move the
positional namespace onto `scope()`, and start v6 with a cold cache in its own
keyspace. There is no runtime v5 shim — the migration is the
rename. If a service can't move yet, keep it on `^5.2`.

## 1. Install

```sh
composer require silviooosilva/cacheer-php:"^6.0"
```

The `^6.0` constraint selects stable 6.x releases and keeps updates within the
same major version.

See [version support](./version-support.md) for the release lines and maintenance policy.


v6 requires PHP 8.3+. The core installs with no backend clients; `ArrayStore` and
`FileStore` work out of the box. Redis and PDO drivers stay optional (Composer
`suggest`).

## 2. Construction: driver selection becomes a named constructor

| v5 | v6 |
|---|---|
| `(new Cacheer())->setDriver()->useFileDriver()` | `Cacheer::file('/var/cache')` |
| `->useDatabaseDriver()` | `Cacheer::database($pdo, 'cacheer')` |
| `->useRedisDriver()` | `Cacheer::redis($connection)` |
| array driver / tests | `Cacheer::inMemory()` |

The database schema is **never** created implicitly — run
`DatabaseStoreSchema::migrate($pdo, $table)` (or `cacheer migrate`) once.

## 3. Method mapping

Every v5 verb below is a method on the cache in v6 — you never reach past it to
the store, and the scope you are in is applied for you.

| v5 | v6 | Notes |
|---|---|---|
| `putCache($k, $v, $ns, $ttl)` | `set($k, $v, $ttl)` | Namespace becomes `scope($ns)->set(...)` |
| `getCache($k, $ns, $ttl)` | `get($k)` | Read-time TTL removed |
| `clearCache($k, $ns)` / `forget()` | `delete($k)` | `scope($ns)->delete(...)` |
| `flushCache()` | `clear()` | Limited to the configured keyspace |
| `forever($k, $v)` | `forever($k, $v)` | Or `set($k, $v, null)` |
| `add($k, $v, $ns, $ttl)` | `add($k, $v, $ttl)` | Lock-serialized where the store can lock |
| `getAndForget()` / `pull()` | `pull($k, $default = null)` | Read and remove in one call |
| `has()` | `has()` | — |
| `missing()` | `missing()` | — |
| `getMany()` / `putMany()` | `many()` / `setMany()` | — |
| positional namespace | `scope('name')` or `in('name')` | Returns a cache of the same type |
| `tag($tag, ...$keys)` | `tag($key, ...$tags)` | Per key; tags are scope-namespaced |
| `flushTag($tag)` | `flushTag($tag)` | Returns how many were removed |
| `increment()` / `decrement()` | `increment()` / `decrement()` | Both kept |
| `renewCache($k, $ttl, $ns)` | `touch($k, $ttl)` | Extends TTL, keeps the value |
| `getAll($ns)` | `entries()` | Scope applied; yields entries with metadata |
| `lock($name, $ttl)` | `lock($name, $ttl)` | Lock names are scope-namespaced |
| `rememberForever()` | `rememberForever()` | — |
| `remember()` / `flexible()` | `remember()` / `flexible()` | Same intent, injected clock |
| `stats()` | `stats()` | Store, scope, policy, real capabilities |
| `useFormatter()` | `formatted()` | An immutable view; base reads stay raw |
| `appendCache()` | read → merge → `set()` | Explicit; wrap in `lock()` if concurrent |
| `isSuccess()` / `getMessage()` | `entry()->isHit()` or return value | Removed from core state |
| static `Cacheer::putCache(...)` | inject a `Cache` instance | No global state in v6 |

The capability-backed rows (`increment`, `touch`, `tag`, `flushTag`, `lock`,
`entries`, `prune`) throw `UnsupportedCapabilityException` on a store that cannot
honor them. Every built-in store honors all of them; if you support pluggable
backends, ask `$cache->supports(AtomicStore::class)` first.

### Automated renames (Rector)

An optional Rector set ships at `rector.php` in the package. It renames the
straightforward v5 methods on `Cacheer` (`putCache`→`set`, `getCache`→`get`,
`renewCache`→`touch`, `getAndForget`→`pull`, …). Verbs v6 kept unchanged — `add`,
`forever`, `missing`, `increment`, `decrement`, `tag`, `flushTag`, `lock`,
`rememberForever`, `stats` — need no rule at all.

It does **not** rewrite construction, move the namespace argument onto `scope()`,
or drop the read-time TTL — do those by hand using the tables above.

```sh
composer require rector/rector --dev
vendor/bin/rector process src --config vendor/silviooosilva/cacheer-php/rector.php --dry-run
```

## 4. Migrating incrementally

There is no runtime v5 shim, but you don't have to convert everything at once:

- Migrate one call site (or module) at a time to the `Cacheer` API; the mapping
  above and the Rector set cover most of the work.
- Keep the **same store** across old and new code during the transition — a
  `Cacheer::file(...)` reads whatever is already on disk (see §5), so migrated and
  unmigrated code share data.
- If a whole service can't move yet, pin it to `^5.2` and migrate it later. v5 and
  v6 are different major lines, not two APIs on one install.

## 5. Cached data: v6 starts cold

v6 does not read v5's cached data. The storage layouts and payload formats differ,
so a v6 store never sees v5 entries: the first v6 request for each key is a miss,
and the value is recomputed and stored in the v6 format. A cache is derived data,
so nothing is lost — expect a warm-up period with more misses, as after a flush.

| Backend | v5 layout | v6 layout |
|---|---|---|
| File | `{dir}/[md5(namespace)/]md5(key).cache` | `{dir}/entries/…` and `{dir}/locks/…` |
| Database | table `cacheer_table` (default) | table `cacheer_store` (default), new schema |
| Redis | `{namespace}[ns:]key`, tags as `tag:…` | `{prefix}:e:…`, `{prefix}:t:…`, `{prefix}:l:…` |

Keep the two keyspaces apart:

- **File**: point v6 at a new directory (sharing works — v6 only touches
  `entries/` and `locks/` — but a separate one makes cleanup trivial).
- **Database**: use a table name different from your v5 table.
- **Redis**: use a v6 `$prefix` that is not your v5 namespace, or another logical
  database.

Once you are past your rollback window, delete the v5 data: the v5 cache
directory, the v5 table, or the v5 Redis keys (by namespace with `SCAN`/`UNLINK`,
or by letting their TTLs expire).

## 6. Database migration and rollback

```php
use Silviooosilva\CacheerPhp\Stores\Support\DatabaseStoreSchema;

DatabaseStoreSchema::migrate($pdo, 'cacheer'); // idempotent
DatabaseStoreSchema::drop($pdo, 'cacheer');    // rollback = drop (cache is derived data)
```

Preview the DDL without executing: `cacheer migrate --dry-run`.

## 7. Verify

- `composer test`, `composer lint`, `composer analyse`
- Re-run your feature tests (framework integrations, Redis command availability)

## 8. Rollback plan

v6 never modifies the v5 keyspace. To fall back: pin `^5.2` again, keep the
previous lock file/vendor directory, and point v5 at its original keyspace. v6 also
never updated or invalidated v5 entries, so **flush the v5 cache when rolling back**
unless values cached before the upgrade are acceptable. Then drop the v6 keyspace
(`cacheer clear --force`).

## Support window

- **v6** is the current stable line and receives features and fixes.
- **v5** receives security and correctness fixes only for 12 months after the 6.0
  stable release. No new features are backported.

> Upgrading from **v4**? First follow the [v5 migration guide](./v5-migration.md),
> then this one.
