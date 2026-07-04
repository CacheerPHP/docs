# Distributed Locks — `Cacheer::lock()`

*New in v5.2.0*

A lock is a named, mutually-exclusive guard backed by the active cache driver. Use it to ensure that only one process runs a critical section at a time — rebuilding a report, sending a one-off email, draining a queue, and so on.

```php
$lock = $cache->lock('rebuild-report', ttl: 30);

if ($lock->acquire()) {
    try {
        rebuildReport();
    } finally {
        $lock->release();
    }
}
```

`lock()` works on an instance **and** through the static facade:

```php
use Silviooosilva\CacheerPhp\Cacheer;

Cacheer::lock('rebuild-report', 30)->get(fn () => rebuildReport());
```

---

## Why & when would I use this?

In production your app is **not one program running once** — it's many PHP
processes at the same time (php-fpm workers serving requests, queue workers, cron
jobs), often across **multiple servers**. None of them know the others exist. A
lock is how those independent processes coordinate through the shared cache
backend so that:

> **only one process runs a block of code at a time — everywhere.**

Reach for a lock whenever *"if two of these ran at once, something bad happens"* —
duplicate work, a duplicated side-effect, or a corrupted result — especially for
a **multi-step** operation (check *then* act) or something that isn't even
database-related (send an email, call an API, generate a file).

### Real-world examples

| Scenario | Without a lock | With a lock |
|----------|----------------|-------------|
| **Cron on multiple servers** (daily digest email) | 3 servers each send it → users emailed 3× | `lock('daily-digest')` — one server wins, others skip |
| **Expensive cache rebuild** (dashboard, search index) | 200 simultaneous misses all recompute → DB melts | one request rebuilds, the rest wait for the result |
| **Payment / order processing** | double-click or retry → charged twice | `lock("invoice:$id")` — charged once |
| **Singleton background worker** | five workers all drain the same queue | whoever holds `lock('queue-drainer')` is "the one" |
| **Fragile external API** ("one request at a time") | concurrent calls collide | calls line up, one at a time |
| **One-time setup on boot** | ten containers all run the migration | it runs once |

> The "expensive cache rebuild" case is so common that CacheerPHP handles it for
> you: [`remember()`](./cache-functions.md#remember--get-or-compute-with-ttl) and
> [`flexible()`](./cache-functions.md#flexible--stale-while-revalidate) already
> use a lock internally to prevent stampedes. You typically only reach for the
> raw `lock()` API for **must-happen-once side-effects** like the rows above.

### When *not* to use it

- **Pure counters** (`+1` to a view count) → use [`increment()`](./cache-functions.md#increment--decrement--numeric-operations), which is already atomic. A lock is overkill.
- **A single row that must be unique** → a database `UNIQUE` constraint is simpler and faster.
- **Plain read caching** → just cache it; the only part that needs protection is the recompute, which `remember()` / `flexible()` handle for you.

---

## `lock()`

```php
$cache->lock(string $name, int $ttl = 60): \Silviooosilva\CacheerPhp\Support\CacheLock
```

Returns a `CacheLock` bound to the active store. `$ttl` is the lock's maximum lifetime in seconds — a safety net so a crashed holder can't block the lock forever.

Each call returns a **fresh** `CacheLock` with its own random *owner token*. A lock can only be released by the holder that acquired it, so an expired-and-reacquired lock is never released out from under its new owner.

> If the active driver does not support locking, `lock()` throws `BadMethodCallException`. All four built-in drivers (File, Database, Redis, Array) support it.

---

## `CacheLock` methods

### `acquire()`

```php
$lock->acquire(): bool
```

Tries to take the lock once, without waiting. Returns `true` if acquired, `false` if it is already held.

### `release()`

```php
$lock->release(): bool
```

Releases the lock — but only if this holder still owns it. Returns `false` if the lock was never held by this instance (or was taken over after expiry).

### `block()`

```php
$lock->block(int $seconds, ?Closure $callback = null): mixed
```

Waits up to `$seconds` for the lock (polling).

- **Without** a callback: returns `true` if the lock was acquired within the window, `false` otherwise.
- **With** a callback: runs it under the lock and releases afterwards, returning the callback's result — or `false` if the lock was never acquired.

```php
// Wait up to 5s, then run exclusively
$result = $cache->lock('export', 30)->block(5, fn () => generateExport());
```

### `get()`

```php
$lock->get(?Closure $callback = null): mixed
```

Like `block()` but **non-blocking** — a single acquire attempt.

- **Without** a callback: same as `acquire()`.
- **With** a callback: runs it under the lock and releases afterwards, returning the result — or `false` if the lock could not be acquired.

```php
$result = $cache->lock('rebuild', 30)->get(fn () => rebuild());
if ($result === false) {
    // someone else is already rebuilding
}
```

### `owner()`

```php
$lock->owner(): string
```

Returns the random owner token identifying this holder. Useful for logging/debugging.

---

## How each driver implements locking

| Driver | Mechanism | Scope |
|--------|-----------|-------|
| **Redis** | `SET key value NX EX ttl`, released with a compare-and-delete Lua script | Cross-process / cross-host |
| **Database** | A `cacheer_locks` table whose `PRIMARY KEY` on the lock name is the atomic gate | Cross-process / cross-host |
| **File** | `flock(LOCK_EX \| LOCK_NB)` on a per-lock file; auto-releases if the process dies | Cross-process (same host) |
| **Array** | In-memory map | Single process only (tests / request-local) |

The Redis, Database, and File drivers provide **real cross-process** mutual exclusion. The Array driver is process-local by nature and is intended for tests or single-request memoisation.

---

## Patterns

### Single-flight (prevent duplicate work)

```php
// Only one worker rebuilds the cache; others move on.
$cache->lock('cache:rebuild', 60)->get(function () use ($cache) {
    $cache->putCache('home', renderHomepage(), ttl: 600);
});
```

### Wait-then-run

```php
// Queue behind whoever holds the lock, up to 10 seconds.
$cache->lock('invoice:'.$id, 30)->block(10, fn () => chargeInvoice($id));
```

### Manual acquire/release

```php
$lock = $cache->lock('migration', 120);

if (! $lock->acquire()) {
    exit("Another migration is already running.\n");
}

try {
    runMigration();
} finally {
    $lock->release();
}
```

---

## Notes

- Lock TTLs are best-effort on the Database and File drivers (they rely on the local clock); pick a `$ttl` comfortably longer than the critical section.
- The File driver keeps a small lock file per lock name under `cacheer-locks/` in the cache directory. These files are reused, not deleted on release — that is intentional and required for correct mutual exclusion.
- Locks share the cache backend but live in a separate keyspace, so they never collide with your cached values.
- Atomic [`increment()` / `decrement()`](./cache-functions.md#increment--decrement--numeric-operations) are built on this same locking layer.
