# Remember, single-flight, and locks

The most common caching pattern is "return it if cached, otherwise compute, store,
and return." `remember()` does exactly that — and protects you from the
**cache stampede** (dogpile) that hits when many requests miss the same key at
once.

## `remember()`

```php
$user = $cache->remember('user:42', ttl: '10 minutes', callback: fn () => $users->find(42));
```

- On a **hit**, the callback never runs.
- On a **miss**, the callback runs once, the result is stored under the TTL, and
  returned.
- A stored `null`/`false`/`0` counts as a hit, so `remember()` won't recompute a
  legitimately empty result every call.

## Single-flight (stampede protection)

When the underlying store implements [`LockingStore`](../api/locks.md) — all four
built-in stores do — `remember()` is **single-flight**:

1. Many workers miss the same key simultaneously.
2. One acquires a short lock and runs the callback.
3. The others block briefly, then read the value the first worker just stored.

So an expensive computation runs **once**, not once per concurrent request. If the
lock can't be acquired within the internal window, a worker falls back to
computing rather than blocking indefinitely — correctness over coordination.

On a store that can't lock, `remember()` degrades to a plain compute-and-store
(no coordination), which is still correct, just not stampede-proof. That holds
through decorators too: wrapping a store in `TieredStore`, `ResilientStore`, or
`InstrumentedStore` never turns a working `remember()` into a failure, because the
kernel asks whether locking is *really* available rather than trusting
`instanceof`.

## `rememberForever()`

For an entry that never expires:

```php
$config = $cache->rememberForever('app:config', fn () => load_config());
$config = $cache->remember('app:config', null, fn () => load_config()); // same thing
```

## Using locks directly

For critical sections that aren't just caching, take a lock yourself:

```php
$lock = $cache->lock('nightly-report', '5 minutes');

if (! $lock->block(10.0)) {
    return; // someone else is running it
}

try {
    generate_report();
} finally {
    $lock->release();
}
```

Locks carry a TTL and self-expire (no deadlock if a holder crashes), and release
is owner-scoped (a lock only deletes its own token). Lock names are namespaced by
scope, so `$cache->in('tenant-a')->lock('import')` and
`$cache->in('tenant-b')->lock('import')` do not contend. See the
[Locks reference](../api/locks.md) for `acquire()`, `block()`, and `release()`.

## Atomic counters

Counters are the store's [`AtomicStore`](../api/drivers.md#capability-interfaces)
capability, called on the cache so the scope applies, and atomic to the guarantee
of the backend:

```php
$next = $cache->increment('page:views');                   // 1, 2, 3, ...
$cache->increment('page:views', 5);
$cache->decrement('stock', 1);
$cache->increment('rate:user:99', 1, initial: 0, ttl: '1 minute');
```

For optimistic concurrency, `compareAndSwap()` is a store-contract primitive:

```php
use Silviooosilva\CacheerPhp\Kernel\Key;

$ok = $cache->store()->compareAndSwap(Key::named('state'), 'idle', 'running');
```

See [Conditional writes & atomic counters](../tutorials/example-14-add-conditional.md).
