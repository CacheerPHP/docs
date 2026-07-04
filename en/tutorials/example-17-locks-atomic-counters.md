# Example 17 — Distributed Locks & Atomic Counters

*New in v5.2.0*

Two related features for safe concurrent access:

- **`lock()`** — a named, driver-backed mutex so only one process runs a critical section at a time.
- **Atomic `increment()` / `decrement()`** — counters that never lose updates under concurrency.

Both work across processes on the File, Database, and Redis drivers.

## Distributed Lock

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useFileDriver();

// Only one process at a time runs the callback; others get `false`.
$result = $cache->lock('rebuild-report', ttl: 30)->get(function () {
    // expensive, must-not-run-twice work
    return rebuildReport();
});

if ($result === false) {
    echo "Another process is already rebuilding the report.\n";
}
```

### Wait for the lock instead of skipping

```php
// Block up to 5 seconds for the lock, then run exclusively.
$charged = $cache->lock("invoice:{$id}", 30)->block(5, fn () => chargeInvoice($id));
```

### Manual acquire / release

```php
$lock = $cache->lock('nightly-job', 120);

if (! $lock->acquire()) {
    exit("Job already running elsewhere.\n");
}

try {
    runNightlyJob();
} finally {
    $lock->release();
}
```

## Atomic Counters

`increment()` and `decrement()` read the current value, change it, and write it back. Across concurrent requests that read-modify-write race and **lose updates**. On any lockable driver, CacheerPHP now serialises each counter update on a per-key lock, so every increment is applied exactly once.

```php
$cache->putCache('views:post:1', 0);

// Safe even when many requests hit this line at the same moment.
$cache->increment('views:post:1');       // +1
$cache->increment('views:post:1', 10);   // +10
$cache->decrement('views:post:1', 3);    // -3
```

The signatures and create-on-miss behaviour are unchanged — only the concurrency guarantee is new:

```php
// Missing key, no default → false, nothing written
$cache->increment('absent');                       // false

// Missing key, with default → created as (default + amount)
$cache->increment('hits', 1, '', 0);               // creates 'hits' = 1
$cache->increment('budget', 10, '', 100);          // missing → 110
$cache->increment('rate', 1, '', 0, '1 hour');     // create-on-miss with 1h TTL
```

## Use Case: Rate Limiting

```php
function tooManyAttempts(Cacheer $cache, string $key, int $max, int $window): bool
{
    // Atomic counter, created on first hit with a sliding TTL window.
    $cache->increment($key, 1, '', 0, $window);
    return (int) $cache->getCache($key) > $max;
}

if (tooManyAttempts($cache, "login:{$ip}", max: 5, window: 60)) {
    http_response_code(429);
    exit("Too many attempts. Try again later.\n");
}
```

## See also

- [Distributed Locks — API reference](../api/locks.md)
- [Cache Functions → increment() / decrement()](../api/cache-functions.md#increment--decrement--numeric-operations)
