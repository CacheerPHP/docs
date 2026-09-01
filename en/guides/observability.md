# Observability — events, metrics, logging

CacheerPHP 6 can tell you exactly what your cache is doing through **typed
events**, an in-process **metrics collector**, and **PSR-3 / PSR-14** bridges —
all without ever recording your cache values by default.

## Instrumenting a cache

Wrap any store with `InstrumentedStore` via the `instrumented` constructor. Every
operation is timed and emitted as a typed [`CacheEvent`](../api/overview.md).

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Observability\{EventBus, MetricsCollector};

$events  = new EventBus();
$metrics = new MetricsCollector();
$events->listen($metrics->record(...));

$cache = Cacheer::instrumented($store, $events);
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

$cache = Cacheer::instrumented($store, new Psr14EventDispatcher($psr14Dispatcher));
```

## The global telemetry tap

Everything above is instance-scoped: a cache emits events only because *you* wrapped
it. `Observability\Telemetry` is the one deliberate exception in v6, and it is worth
being precise about what it is.

- It holds **process-global state** — a static listener list.
- It is **dormant**. With no listeners registered, `Cacheer`'s named constructors
  take the plain, uninstrumented path: no overhead, no behavior change, nothing
  observable.
- **The library registers nothing.** `silviooosilva/cacheer-php` declares only PSR-4
  autoloading, so installing it executes no code.

It exists so a telemetry package can observe caches it did not construct:

```php
use Silviooosilva\CacheerPhp\Observability\Telemetry;

Telemetry::listen(fn ($event) => $collector->record($event));  // opt in
Telemetry::captureValues(true);                                 // off by default
Telemetry::reset();                                             // opt back out
```

Once a listener is registered, every cache built through a named constructor
(`Cacheer::file()`, `::redis()`, …) is transparently instrumented. Caches you
construct directly with `new Cacheer($store)` are never touched by it.

### The autoload-time side effect

Installing [`cacheerphp/monitor`](../cacheer-monitor/index.md) **does** add one:
that package declares `autoload.files`, and its bootstrap registers a listener as
soon as `vendor/autoload.php` is loaded. That is the whole point — zero-wiring
monitoring, see its [quick start](../cacheer-monitor/quick-start.md) — but it is a
real side effect, it comes from that package rather than this one, and you opt into
it by installing it.

If you want no process-global state at all: never call `Telemetry::listen()`, don't
install the monitor, and wire observability explicitly with
`Cacheer::instrumented($store, $events)` as shown at the top of this page.

## Values are never leaked

- Value capture is **off by default**. Events carry the key, timing, byte size,
  and outcome — not the value.
- If you enable capture for debugging, provide a redactor:

  ```php
  $cache = Cacheer::instrumented($store, $events, captureValues: true, redactor: fn ($v) => '[redacted]');
  ```

- Listener failures are **isolated**: `EventBus` wraps every listener in a
  try/catch, so a broken metrics or logging listener can never break a cache
  operation.
