# TTL com DateInterval

Qualquer argumento de TTL aceita um `DateInterval` nativo do PHP.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;

$cache = Cache::inMemory();

$cache->set('a', $v, ttl: new DateInterval('PT30M'));   // 30 minutos
$cache->set('b', $v, ttl: new DateInterval('P1D'));     // 1 dia
$cache->set('c', $v, ttl: new DateInterval('PT1H30M')); // 90 minutos

$cache->remember('d', new DateInterval('PT10M'), fn () => compute());
```

Regras:

- Intervalos com anos ou meses são rejeitados (a duração é ambígua) — use
  dias/horas/minutos/segundos. Um intervalo negativo também é rejeitado.
- Fixe uma expiração absoluta com `Ttl::until()`:

  ```php
  use Silviooosilva\CacheerPhp\Kernel\Ttl;
  use Silviooosilva\CacheerPhp\Support\SystemClock;

  $cache->set('sale', $v, ttl: Ttl::until(new DateTimeImmutable('2026-12-31'), new SystemClock()));
  ```

Veja [TTL e Clock](../api/construtor-de-tempo.md).
