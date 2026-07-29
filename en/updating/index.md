# Upgrading to CacheerPHP 6

CacheerPHP 6 is an instance-first rewrite. You can upgrade in one of two ways:

1. **Bridge first, then modernize.** Swap your v5 object for the `LegacyCacheer`
   bridge — it keeps the v5 method names — then move call sites to the `Cache` API
   at your own pace.
2. **Rewrite directly.** Replace call sites using the mapping below.

Either way, follow one path from installation to a passing test suite.

## 1. Install

```sh
composer require silviooosilva/cacheer-php:^6.0
```

v6 requires PHP 8.3+. The core installs with no backend clients; `ArrayStore` and
`FileStore` work out of the box. Redis and PDO drivers stay optional (Composer
`suggest`).

## 2. Construction: driver selection becomes a named constructor

| v5 | v6 |
|---|---|
| `(new Cacheer())->setDriver()->useFileDriver()` | `Cache::file('/var/cache')` |
| `->useDatabaseDriver()` | `Cache::database($pdo, 'cacheer')` |
| `->useRedisDriver()` | `Cache::redis($connection)` |
| array driver / tests | `Cache::inMemory()` |

The database schema is **never** created implicitly — run
`DatabaseStoreSchema::migrate($pdo, $table)` (or `cacheer migrate`) once.

## 3. Method mapping

| v5 | v6 | Notes |
|---|---|---|
| `putCache($k, $v, $ns, $ttl)` | `set($k, $v, $ttl)` | Namespace becomes `scope($ns)->set(...)` |
| `forever($k, $v)` | `set($k, $v, null)` | `null` TTL = forever |
| `getCache($k, $ns, $ttl)` | `get($k)` | Read-time TTL removed |
| `clearCache($k, $ns)` | `delete($k)` | `scope($ns)->delete(...)` |
| `flushCache()` | `clear()` | Limited to the configured keyspace |
| `getAndForget()` / `pull()` | `pull()` (bridge) | Atomicity reported by capability |
| `has()` / `missing()` | `has()` | — |
| positional namespace | `scope('name')` | Returns a scoped cache |
| `tag($tag, ...$keys)` | `TaggableStore::tag()` | Capability, not core |
| `increment()` / `decrement()` | `AtomicStore::increment()` | Capability, not core |
| `isSuccess()` | `entry()->isHit()` or return value | Removed from core state |
| `remember()` / `flexible()` | `remember()` / `flexible()` | Same intent, injected clock |

### Automated renames (Rector)

An optional Rector set ships at `rector.php` in the package. It renames the
straightforward v5 methods on `Cacheer`/`LegacyCacheer`. It does **not** rewrite
construction, move the namespace argument onto `scope()`, or drop the read-time
TTL — do those by hand using the tables above.

```sh
composer require rector/rector --dev
vendor/bin/rector process src --config vendor/silviooosilva/cacheer-php/rector.php --dry-run
```

## 4. The compatibility bridge

The bridge is a drop-in for the v5 surface on top of the v6 engine:

```php
use Silviooosilva\CacheerPhp\Compat\LegacyCacheer;

$cache = LegacyCacheer::file('/var/cache');   // or ::inMemory()
$cache->putCache('user:1', $user, 'accounts', 3600);
$user = $cache->getCache('user:1', 'accounts');
```

Enable deprecations in development to locate call sites to migrate — they are
**silent by default** so production logs stay clean:

```php
$cache = LegacyCacheer::file('/var/cache', emitDeprecations: true);
```

The bridge also exposes `forever`, `has`, `missing`, `pull`/`getAndForget`,
`renewCache`, `increment`/`decrement`, `remember`/`rememberForever`, `tag`/
`flushTag`, `appendCache`, and `isSuccess`/`getMessage`.

## 5. Data compatibility and rewrite-on-read

v6 writes an authenticated, versioned envelope but can still **read** values
written by v5. Construct the store's pipeline with a `V5PayloadReader` matching the
compression/encryption your v5 app used (v5 payloads are not self-describing), and
opt into rewrite-on-read to re-encode legacy values in the v6 envelope on read:

```php
use Silviooosilva\CacheerPhp\Config\PipelineConfig;
use Silviooosilva\CacheerPhp\Storage\Compat\V5PayloadReader;
use Silviooosilva\CacheerPhp\Stores\FileStore;

$pipeline = PipelineConfig::default()->withV5Reader(new V5PayloadReader(compression: true));
$store = new FileStore('/var/cache', $pipeline->codec(), migrateLegacyOnRead: true);
```

- v5's AES-256-**CBC** payloads are unauthenticated; a wrong key or tampering
  surfaces only as a failed `unserialize`, never cryptographically. New writes
  always use the authenticated v6 envelope.
- `FileStore` and `DatabaseStore` support rewrite-on-read; Redis entries migrate on
  their next write (legacy reads keep working until then).

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

Rewrite-on-read is opt-in, so a read-only rollout never mutates v5 data. To fall
back: pin `^5.2` again, keep the previous lock file/vendor directory, and clear any
v6-only envelopes (`cacheer clear --force`).

## Support window

- **v6** is the actively developed line.
- **v5** receives security and correctness fixes only for 12 months after the 6.0
  stable release.

> Upgrading from **v4**? First follow the [v5 migration guide](./v5-migration.md),
> then this one.
