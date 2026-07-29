# Locks

A store that implements `LockingStore` can hand out named, TTL-bounded locks.
The kernel uses them internally for single-flight [`remember()`](./cache-functions.md#remember)
and stale refresh; you can use them directly for any critical section.

## `LockingStore`

```php
namespace Silviooosilva\CacheerPhp\Contracts;

interface LockingStore
{
    public function lock(string $name, Ttl $ttl): Lock;
}
```

All four built-in stores implement it. The *scope* of the guarantee differs:
`ArrayStore` is in-process only; `FileStore` is safe across processes on one host
(`flock`); `DatabaseStore` uses a locks row; `RedisStore` uses `SET NX` with a
compare-and-delete (Lua) release.

## `Lock`

```php
namespace Silviooosilva\CacheerPhp\Contracts;

interface Lock
{
    public function acquire(): bool;        // non-blocking; true if acquired
    public function block(float $seconds): bool;  // wait up to N seconds
    public function release(): bool;        // release only if we still own it
}
```

- A lock carries a **TTL** and self-expires, so a crashed holder never deadlocks
  the keyspace.
- Release is **owner-scoped** (compare-and-delete): a lock only deletes its own
  token, so a slow holder cannot release a lock another worker has since acquired.

## Using a lock

```php
use Silviooosilva\CacheerPhp\Kernel\Ttl;

$lock = $store->lock('import:catalog', Ttl::seconds(30));

if ($lock->acquire()) {
    try {
        run_import();
    } finally {
        $lock->release();
    }
}
```

Blocking with a timeout:

```php
$lock = $store->lock('nightly-job', Ttl::minutes(5));

if (! $lock->block(10.0)) {
    return; // another worker holds it; skip this run
}

try {
    run_job();
} finally {
    $lock->release();
}
```

## Single-flight `remember()`

When the store implements `LockingStore`, `remember()` is stampede-safe
automatically: on a concurrent miss, one caller acquires the lock and computes
while the others block briefly and then read the freshly-stored value. If the lock
cannot be acquired within the internal window, callers fall back to computing
rather than blocking forever. See the
[Remember & locks guide](../guides/remember-and-locks.md).
