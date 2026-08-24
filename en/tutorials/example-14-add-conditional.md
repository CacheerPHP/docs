# Conditional writes & atomic counters

## `add()` — store only if absent

`add()` writes only when the key is missing and tells you whether *this* call was
the one that stored it. When the store can lock, the check and the write are
serialized, so it is a sound first-writer-wins across processes — not the racy
`has()` + `set()` you would otherwise hand-roll.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

if ($cache->add('import:running', 1, ttl: 300)) {
    // we won — run the import
} else {
    // someone else is already on it
}
```

A falsy stored value is still a value, so `add()` will not overwrite a stored
`false`, `0`, or `''`.

## Counters — `increment()` / `decrement()`

Counters are the `AtomicStore` capability, atomic to the guarantee of the backend,
and called on the cache so the scope applies.

```php
// increment(string|Key $key, int $amount = 1, ?int $initial = null, $ttl = null): int
$cache->increment('page:views');                      // 1 (key must exist without $initial)
$cache->increment('page:views', 5);                   // 6
$cache->increment('score', 1, initial: 100);          // starts at 100 + 1 = 101
$cache->decrement('stock', 1);                        // reads better than a negative amount

// Time-bounded counter — a rate-limit window.
$cache->increment('rate:user:99', 1, initial: 0, ttl: '1 minute');
```

Counters respect scope, so tenants cannot collide:

```php
$cache->in('tenant-a')->increment('signups', 1, initial: 0);  // 1
$cache->in('tenant-b')->increment('signups', 5, initial: 0);  // 5
```

## Once-only side effects — `lock()`

`add()` protects a cache entry; a lock protects your *work*. Use one when the
side effect must happen exactly once.

```php
$lock = $cache->lock('job:send_invoice:42', 60);
if ($lock->acquire()) {
    try {
        $invoices->send(42);
    } finally {
        $lock->release();
    }
}
```

## Compare-and-swap

For optimistic concurrency, `compareAndSwap()` writes only if the current value
still matches what you last read. It is a store-contract primitive, reached
through `store()`:

```php
use Silviooosilva\CacheerPhp\Kernel\Key;

// compareAndSwap(Key, mixed $expected, mixed $value, ?Ttl = null): bool
$ok = $cache->store()->compareAndSwap(Key::named('job:state'), 'idle', 'running');
```

Guarantees per backend: `ArrayStore` is atomic within one process; `FileStore`
serializes on a per-key file lock; `DatabaseStore` uses a row-locked
read-modify-write; `RedisStore` uses server-side atomics. See
[Remember & locks](../guides/remember-and-locks.md#atomic-counters).
