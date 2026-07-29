# Observabilidade — eventos, métricas, logging

O CacheerPHP 6 pode dizer exatamente o que o seu cache está fazendo por meio de
**eventos tipados**, um **coletor de métricas** em processo e pontes **PSR-3 / PSR-14**
— tudo sem nunca registrar seus valores de cache por padrão.

## Instrumentando um cache

Embrulhe qualquer store com `InstrumentedStore` via o construtor `instrumented`. Toda
operação é cronometrada e emitida como um [`CacheEvent`](../api/visao-geral.md) tipado.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Observability\{EventBus, MetricsCollector};

$events  = new EventBus();
$metrics = new MetricsCollector();
$events->listen($metrics->record(...));

$cache = Cache::instrumented($store, $events);
```

## Tipos de evento

`CacheEventType` cobre: `Hit`, `Miss`, `Write`, `Delete`, `Clear`, `Prune`, `Failure`,
`Promotion`, `StaleServed`, `Refresh` e `LockContended` (valores como `cache.hit`).
Eventos de nível de núcleo (promoção, stale-served, refresh, contenção de lock) fluem
pelo mesmo dispatcher quando você o passa aos caches em camadas.

## Métricas

`MetricsCollector::snapshot()` retorna um array simples para logar ou exportar:

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

## Logging PSR-3

`PsrLoggerSubscriber` transforma eventos em registros de log estruturados —
**apenas metadados**, nunca valores. Falhas logam em `WARNING`; o resto em `DEBUG`.

```php
use Silviooosilva\CacheerPhp\Observability\PsrLoggerSubscriber;

$events->listen(new PsrLoggerSubscriber($psrLogger));
```

## Dispatch PSR-14

Encaminhe eventos de cache para um dispatcher PSR-14 existente:

```php
use Silviooosilva\CacheerPhp\Observability\Psr14EventDispatcher;

$cache = Cache::instrumented($store, new Psr14EventDispatcher($psr14Dispatcher));
```

## Valores nunca vazam

- A captura de valores está **desligada por padrão**. Eventos carregam a chave, o
  tempo, o tamanho em bytes e o desfecho — não o valor.
- Se você habilitar captura para depurar, forneça um redator:

  ```php
  $cache = Cache::instrumented($store, $events, captureValues: true, redactor: fn ($v) => '[redacted]');
  ```

- Falhas de listeners são **isoladas**: o `EventBus` embrulha cada listener em um
  try/catch, então um listener de métricas ou logging quebrado nunca quebra uma
  operação de cache.
