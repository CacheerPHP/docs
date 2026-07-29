# Cache em camadas (L1/L2)

Coloque uma store local rápida na frente de uma compartilhada. As leituras batem na
memória local primeiro e caem para a rede só num miss; o valor é promovido de volta ao
L1.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Kernel\Ttl;
use Silviooosilva\CacheerPhp\Stores\ArrayStore;
use Silviooosilva\CacheerPhp\Stores\Support\PredisConnection;
use Silviooosilva\CacheerPhp\Stores\RedisStore;
use Silviooosilva\CacheerPhp\Support\SystemClock;

$clock = new SystemClock();
$l2 = new RedisStore(new PredisConnection(new Predis\Client()), 'app', clock: $clock);

$cache = Cache::tiered(
    l1: new ArrayStore($clock),  // por processo, instantâneo
    l2: $l2,                     // compartilhado na frota
    l1MaxTtl: Ttl::seconds(10),  // limita quanto tempo um valor vive localmente
);

$user = $cache->remember('user:42', '10 minutes', fn () => $users->find(42));
// primeira chamada: miss nos dois, calculado, gravado no L2 + L1
// próximas chamadas neste processo: servidas pelo L1, sem ida ao Redis
```

- Escritas e deletes vão para **ambas** as camadas, para não divergirem.
- Um token de geração compartilhado mantém workers de vida longa coerentes quando outro
  worker invalida o L2.
- Camadas otimiza **velocidade**, não disponibilidade — um miss no L2 é um miss real.
  Para quedas, veja a [Store resiliente](../guias/store-resiliente.md).

Veja o [guia de Cache em camadas](../guias/cache-em-camadas.md).
