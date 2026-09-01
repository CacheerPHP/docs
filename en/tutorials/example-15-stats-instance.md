# Observability: events & metrics

> v5's `stats()` reported driver flags. v6 keeps `stats()` — now describing the
> cache itself — and adds typed events plus an in-process metrics collector,
> without ever recording your cache values.

## `stats()` — what this cache *is*

```php
$cache->in('reports')->withPolicy($policy)->stats();
// [
//     'store'        => 'FileStore',
//     'scope'        => 'reports',
//     'policy'       => true,
//     'capabilities' => ['batch' => true, 'atomic' => true, 'locking' => true, ...],
// ]
```

Capabilities are reported honestly, decorators included — this is the same answer
`supports()` gives, so it is safe to branch on. Nothing here contains cached
values, so it is fine to log or expose on a health endpoint.

## Events & metrics — what it has been *doing*

Wrap a store with `instrumented`, attach a `MetricsCollector`, and read a snapshot.

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Observability\{EventBus, MetricsCollector};
use Silviooosilva\CacheerPhp\Stores\ArrayStore;
use Silviooosilva\CacheerPhp\Support\SystemClock;

$events  = new EventBus();
$metrics = new MetricsCollector();
$events->listen($metrics->record(...));

$cache = Cacheer::instrumented(new ArrayStore(new SystemClock()), $events);

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
