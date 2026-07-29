# Observabilidade: eventos e métricas

> O `stats()` da v5 reportava flags do driver. A v6 dá eventos tipados e um coletor de
> métricas em processo — sem nunca registrar seus valores de cache.

Embrulhe uma store com `instrumented`, anexe um `MetricsCollector` e leia um snapshot.

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

Logue os mesmos eventos (apenas metadados) via PSR-3:

```php
use Silviooosilva\CacheerPhp\Observability\PsrLoggerSubscriber;

$events->listen(new PsrLoggerSubscriber($psrLogger));
```

A captura de valores está desligada por padrão, e falhas de listeners não podem quebrar
uma operação de cache. Veja o [guia de Observabilidade](../guias/observabilidade.md).
