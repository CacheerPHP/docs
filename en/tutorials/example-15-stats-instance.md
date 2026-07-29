# Observability: events & metrics

> v5's `stats()` reported driver flags. v6 gives you typed events and an
> in-process metrics collector — without ever recording your cache values.

Wrap a store with `instrumented`, attach a `MetricsCollector`, and read a snapshot.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Observability\{EventBus, MetricsCollector};
use Silviooosilva\CacheerPhp\Stores\ArrayStore;
use Silviooosilva\CacheerPhp\Support\SystemClock;

$events  = new EventBus();
$metrics = new MetricsCollector();
$events->listen($metrics->record(...));

$cache = Cache::instrumented(new ArrayStore(new SystemClock()), $events);

$cache->set('a', 1);
$cache->get('a');       // hit
$cache->get('missing'); // miss

$metrics->snapshot();
// ['hits' => 1, 'misses' => 1, 'hit_rate' => 0.5, 'writes' => 1, 'bytes_written' => ..., 'avg_micros' => ...]
```

Log the same events (metadata only) via PSR-3:

```php
use Silviooosilva\CacheerPhp\Observability\PsrLoggerSubscriber;

$events->listen(new PsrLoggerSubscriber($psrLogger));
```

Value capture is off by default, and listener failures can't break a cache
operation. See the [Observability guide](../guides/observability.md).
