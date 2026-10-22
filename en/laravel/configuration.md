# Configure CacheerPHP for Laravel

Each `cacheer` store is an entry in `config/cache.php`. You can define several, for example a Redis store as the default and a file store for local data.

## Options

| Option | Default | Description |
| --- | --- | --- |
| `driver` | — | Always `cacheer`. |
| `backend` | `file` | Where entries are stored: `file`, `array`, `database`, or `redis`. |
| `path` | `storage/framework/cache/cacheer` | Directory for the `file` backend. |
| `connection` | `default` for redis; the default connection for database | A connection from `config/database.php`. |
| `table` | `cacheer_store` | Table for the `database` backend. |
| `redis_prefix` | `cacheer` | Key prefix inside Redis for the `redis` backend. |
| `prefix` | `cache.prefix` | The store's isolated keyspace. See [prefix](#prefix). |
| `fallback` | none | A backend to serve from while the primary is down: a backend name such as `'file'`, or a full backend array. |
| `l1` | none | Keep in-process copies in front of the store: a number of seconds (their maximum age), or `true` for 60. |
| `encryption` | none | `['active' => id, 'passphrases' => [id => passphrase]]`. |
| `compress` | `false` | `true`, or a gzip level from 1 to 9. |
| `max_value_bytes` | none | Reject larger values on write and on read. |

## Backends

```php
// Files on this host, no services needed.
'cacheer' => ['driver' => 'cacheer', 'backend' => 'file'],

// A Laravel redis connection.
'cacheer' => ['driver' => 'cacheer', 'backend' => 'redis', 'connection' => 'cache'],

// A Laravel database connection. Run the migration first (see installation).
'cacheer' => ['driver' => 'cacheer', 'backend' => 'database', 'connection' => 'mysql'],

// Process memory, for tests.
'cacheer' => ['driver' => 'cacheer', 'backend' => 'array'],
```

The redis backend reads the host, port, password, and database of the Laravel connection you name, and opens its own connection with them. It does not use Laravel's redis `prefix`, `serializer`, or `compression` options: CacheerPHP prefixes and serializes on its own, and a client that rewrites keys or values would break its stored entries. Both the `phpredis` and `predis` clients work; Redis clusters are not supported.

## Failover

Serve from another backend while Redis is down:

```php
'cacheer' => [
    'driver' => 'cacheer',
    'backend' => 'redis',
    'connection' => 'cache',
    'fallback' => 'file',
],
```

Reads and writes move to the fallback when Redis fails, and go back to Redis when it recovers. Counters and locks never fall back: during the outage they fail closed (a lock is not acquired) rather than hand out a second, unshared lock.

Failover state lives in each PHP process. A long-running process (an Octane worker, a queue worker) remembers the keys it changed during the outage and invalidates them on Redis before using it again. A process that ends does not: under PHP-FPM every request is its own process, so a value changed or deleted during an outage can be served again from Redis after it recovers, until it expires. If that matters, keep TTLs short for such data, or run `php artisan cache:clear` once Redis is back. See [resilient stores](../guides/resilient-store.md) for how the circuit breaker works.

## Tiered caching

Keep recent values in process memory in front of the shared store:

```php
'cacheer' => [
    'driver' => 'cacheer',
    'backend' => 'redis',
    'connection' => 'cache',
    'l1' => 60,
],
```

This pays off in long-running workers, such as Octane and queue workers; under PHP-FPM the copies last for one request. `Cache::flush()` reaches every worker at its next coherence check, at most five seconds later. A single-key change made by another worker is seen within the `l1` age at most. Locks always read the shared store, never a local copy. See [tiered caching](../guides/tiered-caching.md).

## Encryption and compression

```php
'cacheer' => [
    'driver' => 'cacheer',
    'backend' => 'redis',
    'connection' => 'cache',
    'encryption' => [
        'active' => 'v1',
        'passphrases' => ['v1' => env('CACHEER_KEY')],
    ],
    'compress' => true,
],
```

Values are encrypted with AES-256-GCM and authenticated before they are read. Keys are not encrypted, so do not put secrets in cache keys. To rotate, add a new passphrase, make it `active`, and keep the old one until its entries have expired. An empty passphrase, such as an unset `CACHEER_KEY`, is refused when the store is created rather than used. See [encryption and compression](../guides/encryption-and-compression.md).

## Prefix

Like Laravel's own stores, a `cacheer` store uses its `prefix` option, or else `cache.prefix`. The prefix becomes the store's isolated keyspace (a CacheerPHP [scope](../guides/scopes.md)):

- `Cache::flush()` clears that keyspace only. Other applications and stores sharing the backend keep their entries.
- Matching is literal: flushing `app_1` never touches `app_10`.

With an empty prefix, `Cache::flush()` clears the whole backend keyspace of that store.

## Clock

Entries expire on Laravel's clock, so `$this->travel()` and `Carbon::setTestNow()` affect them. To supply a clock of your own, bind `Silviooosilva\CacheerPhp\Contracts\Clock` in the container.
