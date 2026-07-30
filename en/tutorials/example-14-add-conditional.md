# Atomic counters & compare-and-swap

Counters and conditional writes are a store capability (`AtomicStore`), atomic to
the guarantee of the backend.

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Kernel\Key;
use Silviooosilva\CacheerPhp\Stores\FileStore;

$store = new FileStore(__DIR__ . '/cache');

// increment(Key, int $amount = 1, ?int $initial = null, ?Ttl $ttl = null): int
$store->increment(Key::named('page:views'));            // 1
$store->increment(Key::named('page:views'), amount: 5); // 6
$store->increment(Key::named('score'), initial: 100);   // starts at 100 + 1 = 101

// Decrement is a negative increment.
$store->increment(Key::named('stock'), amount: -1);
```

Compare-and-swap writes only if the current value matches — a lock-free way to
guard a transition:

```php
// compareAndSwap(Key, mixed $expected, mixed $value, ?Ttl = null): bool
$ok = $store->compareAndSwap(Key::named('job:state'), 'idle', 'running');
if ($ok) {
    // we won the race; run the job
}
```

Guarantees per backend: `ArrayStore` is atomic within one process; `FileStore`
serializes on a per-key file lock; `DatabaseStore` uses a row-locked
read-modify-write; `RedisStore` uses server-side atomics. See
[Remember & locks](../guides/remember-and-locks.md#atomic-counters).
