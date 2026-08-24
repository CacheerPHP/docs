# Locks & critical sections

A store that implements `LockingStore` (all four built-ins) hands out named,
TTL-bounded locks so only one worker runs a critical section at a time.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

$lock = $cache->lock('nightly-report', '5 minutes');

// Wait up to 10s for the lock; skip the run if someone else holds it.
if (! $lock->block(10.0)) {
    return;
}

try {
    generate_report();
} finally {
    $lock->release();
}
```

Non-blocking variant:

```php
$lock = $cache->lock('import', 30);

if ($lock->acquire()) {          // true only if we got it right now
    try {
        run_import();
    } finally {
        $lock->release();
    }
}
```

- Locks carry a **TTL** and self-expire, so a crashed holder never deadlocks the
  keyspace.
- Release is **owner-scoped**: a lock only deletes its own token.
- Lock names are **namespaced by scope** — `$cache->in('tenant-a')->lock('import')`
  and `$cache->in('tenant-b')->lock('import')` never contend.
- `remember()` uses these locks internally for
  [stampede protection](./example-18-stampede-and-swr.md).

See the [Locks reference](../api/locks.md).
