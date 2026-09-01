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
- Lock names are **namespaced by scope**, so two scopes asking for the same name
  get independent mutexes.

## Using a lock

Take locks on the **cache** — the name is namespaced by scope and the capability
is checked for you:

```php
public function lock(string $name, Ttl|DateInterval|int|string $ttl = 60): Lock
```

```php
$lock = $cache->lock('import:catalog', 30);

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
$lock = $cache->lock('nightly-job', '5 minutes');

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
rather than blocking forever.

When the store *cannot* lock — including a decorator around a store that cannot —
`remember()` degrades to a plain compute rather than failing. The kernel decides
this with `Capabilities::supports()`, never `instanceof`; see
[Stores & capabilities](./drivers.md). See also the
[Remember & locks guide](../guides/remember-and-locks.md).
