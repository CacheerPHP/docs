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
(no coordination), which is still correct, just not stampede-proof.

## `rememberForever`

Pass `null` as the TTL for an entry that never expires:

```php
$config = $cache->remember('app:config', ttl: null, callback: fn () => load_config());
```

## Using locks directly

For critical sections that aren't just caching, take a lock yourself:

```php
use Silviooosilva\CacheerPhp\Kernel\Ttl;

$lock = $store->lock('nightly-report', Ttl::minutes(5));

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
is owner-scoped (a lock only deletes its own token). See the
[Locks reference](../api/locks.md) for `acquire()`, `block()`, and `release()`.

## Atomic counters

Counters are a capability of the store ([`AtomicStore`](../api/drivers.md#capability-interfaces)),
atomic to the guarantee of the backend:

```php
use Silviooosilva\CacheerPhp\Kernel\Key;

$next = $store->increment(Key::named('page:views'));           // 1, 2, 3, ...
$store->increment(Key::named('page:views'), amount: 5);
$ok = $store->compareAndSwap(Key::named('state'), 'idle', 'running');
```

See [Atomic counters & CAS](../tutorials/example-14-add-conditional.md).
