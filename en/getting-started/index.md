# Getting Started

CacheerPHP 6 is an instance-first cache: a small `Cacheer` kernel over a minimal
`Store` contract, with optional capabilities (batch, tags, locks, atomic
counters), composable decorators (tiered, resilient, instrumented), a versioned
authenticated storage pipeline, and PSR-16/PSR-6 adapters.

## Requirements

| Requirement | Details |
|-------------|---------|
| **PHP** | 8.3 or newer |
| **Optional** | `ext-openssl` (AES-256-GCM encryption), `ext-zlib` (gzip) |
| **Optional** | `ext-pdo` + a PDO driver for the database store |
| **Optional** | `predis/predis` or `ext-redis` for the Redis store |

The core installs and runs with the Array and File stores using **none** of the
optional pieces.

## Installation

```sh
composer require silviooosilva/cacheer-php
```

This pulls in the PSR contracts automatically (`psr/simple-cache`, `psr/cache`,
`psr/log`, `psr/event-dispatcher`).

## Quick Start

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

// Dependency-free, in-process (great for tests and short CLI runs).
$cache = Cacheer::inMemory();

// Write with a TTL: seconds, "10 minutes", a DateInterval, or null (forever).
$cache->set('user:123', ['name' => 'John Doe'], ttl: '10 minutes');

// Read (returns the default on a miss).
$user = $cache->get('user:123', default: null);

// Compute-once: run the callback on a miss, store and return it.
$report = $cache->remember('report:daily', ttl: 3600, callback: fn () => build_report());

// Existence and deletion.
$cache->has('user:123');   // true
$cache->delete('user:123');
```

## Choosing a store

Swap the store, keep the API. See [Stores & capabilities](../api/drivers.md).

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::inMemory();                    // in-process array store
$cache = Cacheer::file('/var/cache/app');        // persistent, dependency-free
$cache = Cacheer::database($pdo, 'cacheer');     // inject your own PDO
$cache = Cacheer::redis($connection);            // predis or phpredis adapter
```

Create the database schema explicitly first (it is never a side effect):

```php
use Silviooosilva\CacheerPhp\Stores\Support\DatabaseStoreSchema;

DatabaseStoreSchema::migrate($pdo, 'cacheer');
```

## Scopes

Scopes replace stringly namespaces with isolated keyspaces you can clear on their
own. See the [Scopes guide](../guides/scopes.md).

```php
$cache->scope('reports')->set('daily', $rows);
$cache->in('billing')->set('daily', $invoice);    // in() is an alias of scope()
$cache->scope('reports')->clear();                // clears only that scope
```

Scoping returns another `Cacheer`, so a scoped cache keeps the whole API — and the
scope applies to counters, tags, and locks too, not just `get`/`set`.

## Counters, tags, locks, TTL renewal

Capabilities are implemented by the store and called on the cache, so this cache's
scope is applied for you:

```php
$cache->increment('visits');                  // atomic counter
$cache->decrement('stock', 5, initial: 100);
$cache->touch('session:1', '1 hour');         // renew a TTL, keep the value
$cache->tag('product:1', 'products');
$cache->flushTag('products');
$lock = $cache->lock('nightly-import', 300);

// Pluggable backend? Ask first — the answer is honest through decorators too.
$cache->supports(\Silviooosilva\CacheerPhp\Contracts\AtomicStore::class);
```

## Everyday verbs

```php
$cache->forever('config', $config);                     // no expiry
$cache->add('lock:job', 1, ttl: 60);                    // store only if absent
$cache->pull('flash');                                  // read once, then remove
$cache->missing('user:1');                              // inverse of has()
$cache->rememberForever('version', fn () => compute());
$cache->stats();                                        // store, scope, capabilities
```

## Compute-once and stale-while-revalidate

```php
// Runs the callback once even under a concurrent stampede.
$user = $cache->remember('user:42', '10 minutes', fn () => $users->find(42));

// Serve fresh for 30s, then serve stale while one worker refreshes, up to 300s.
$feed = $cache->flexible('feed', fresh: 30, stale: 300, callback: fn () => build_feed());
```

See [Remember & locks](../guides/remember-and-locks.md) and
[Stale-while-revalidate](../guides/stale-while-revalidate.md).

## TTL formats

A TTL can be an int (seconds), a human string, a `DateInterval`, or `null`
(forever). See [TTL & Clock](../api/time-builder.md).

```php
$cache->set('key', $data, ttl: 3600);
$cache->set('key', $data, ttl: '2 hours');
$cache->set('key', $data, ttl: new \DateInterval('PT30M'));
$cache->set('key', $data, ttl: null); // forever
```

## PSR adapters

```php
use Silviooosilva\CacheerPhp\Psr\{Psr16Cache, Psr6Pool};
use Silviooosilva\CacheerPhp\Support\SystemClock;

$psr16 = new Psr16Cache($cache);                    // Psr\SimpleCache\CacheInterface
$pool  = new Psr6Pool($cache, new SystemClock());   // Psr\Cache\CacheItemPoolInterface
```

See [PSR-16 & PSR-6 adapters](../api/psr16-adapter.md).

## Coming from v5?

Migrating is mostly mechanical — rename the v5 methods to the v6 names
(`putCache`→`set`, `getCache`→`get`, positional namespace → `scope()`), and let
your existing cached data upgrade itself via rewrite-on-read. An optional Rector
set automates the common renames; if a service can't move yet, keep it on `^5.2`.

See the [migration guide](../updating/index.md) for the full mapping.

## Next Steps

- [Guides](../guides/configuration.md) — feature deep dives
- [API Reference](../api/index.md) — complete method documentation
- [Tutorials](../tutorials/index.md) — task-focused examples
- [Migration guide](../updating/index.md) — upgrading from v5
