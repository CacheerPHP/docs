# Observability — events, metrics, logging

CacheerPHP 6 can tell you exactly what your cache is doing through **typed
events**, an in-process **metrics collector**, and **PSR-3 / PSR-14** bridges —
all without ever recording your cache values by default.

## Instrumenting a cache

Wrap any store with `InstrumentedStore` via the `instrumented` constructor. Every
operation is timed and emitted as a typed [`CacheEvent`](../api/overview.md).

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Observability\{EventBus, MetricsCollector};

$events  = new EventBus();
$metrics = new MetricsCollector();
$events->listen($metrics->record(...));

$cache = Cache::instrumented($store, $events);
```

## Event types

`CacheEventType` covers: `Hit`, `Miss`, `Write`, `Delete`, `Clear`, `Prune`,
`Failure`, `Promotion`, `StaleServed`, `Refresh`, and `LockContended` (string
values like `cache.hit`). Kernel-level events (promotion, stale-served, refresh,
lock contention) flow through the same dispatcher when you pass it to
`tiered()` / `flexible()`-capable caches.

## Metrics

`MetricsCollector::snapshot()` returns a plain array you can log or export:

```php
$metrics->snapshot();
// [
//   'hits' => 812, 'misses' => 44, 'hit_rate' => 0.9486,
//   'writes' => 60, 'deletes' => 3, 'failures' => 0,
//   'promotions' => 12, 'stale_served' => 5, 'refreshes' => 5,
//   'lock_contended' => 1, 'bytes_written' => 148213,
//   'avg_micros' => 41.2, 'max_micros' => 900.0,
// ]
```

## PSR-3 logging

`PsrLoggerSubscriber` turns events into structured log records — **metadata only**,
never values. Failures log at `WARNING`; everything else at `DEBUG`.

```php
use Silviooosilva\CacheerPhp\Observability\PsrLoggerSubscriber;

$events->listen(new PsrLoggerSubscriber($psrLogger));
```

## PSR-14 dispatching

Bridge cache events onto an existing PSR-14 event dispatcher:

```php
use Silviooosilva\CacheerPhp\Observability\Psr14EventDispatcher;

$cache = Cache::instrumented($store, new Psr14EventDispatcher($psr14Dispatcher));
```

## Values are never leaked

- Value capture is **off by default**. Events carry the key, timing, byte size,
  and outcome — not the value.
- If you enable capture for debugging, provide a redactor:

  ```php
  $cache = Cache::instrumented($store, $events, captureValues: true, redactor: fn ($v) => '[redacted]');
  ```

- Listener failures are **isolated**: `EventBus` wraps every listener in a
  try/catch, so a broken metrics or logging listener can never break a cache
  operation.
