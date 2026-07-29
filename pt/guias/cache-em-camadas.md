# Cache em camadas (L1/L2)

Um cache em camadas coloca uma **store local rápida (L1)** na frente de uma **store
compartilhada (L2)**. As leituras batem na memória local primeiro e só caem para a
rede num miss; o valor é então promovido de volta ao L1, para a próxima leitura ser
rápida.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Stores\ArrayStore;

$cache = Cache::tiered(
    l1: new ArrayStore($clock),   // por processo, leituras instantâneas
    l2: $redisStore,              // compartilhado na frota
);
```

## Read-through e promoção

1. `get()` checa o L1. Hit → retorna.
2. Miss → checa o L2. Hit → **promove** ao L1, depois retorna.
3. Miss nos dois → um miss real.

Leituras seguintes no mesmo processo são servidas pelo L1 sem tocar no L2.

## Escritas, deletes, clears

Escritas e deletes vão para **ambas** as camadas para não divergirem:

- `set()` grava no L2 e no L1 (o TTL do L1 é limitado — veja abaixo).
- `delete()` remove das duas.
- `clear()` limpa as duas.

## Limitando o TTL do L1

O L1 é um espelho quente, não a fonte da verdade. Limite quanto tempo um valor pode
viver localmente para que uma entrada de vida longa no L2 não fique presa em memória
local stale:

```php
use Silviooosilva\CacheerPhp\Kernel\Ttl;

$cache = Cache::tiered($l1, $l2, l1MaxTtl: Ttl::seconds(10));
```

## Coerência entre workers

O L1 vive dentro de um único worker, então quando outro worker invalida dados no L2,
o L1 deste worker poderia ainda ter uma cópia stale. `TieredStore` usa um **token de
geração** compartilhado: invalidações em massa incrementam a geração, e uma entrada
de L1 de uma geração antiga é tratada como miss. Workers de vida longa, portanto,
captam invalidações sem reiniciar.

## Camadas não é resiliência

Um cache em camadas otimiza **performance**: um miss no L2 é um miss genuíno. Se você
quer continuar servindo quando uma store está **fora do ar**, isso é outra
ferramenta — [`ResilientStore`](./store-resiliente.md). As duas se compõem.

## Observabilidade

Promoções são emitidas como eventos `cache.promotion` quando você embrulha o cache em
camadas com instrumentação:

```php
$cache = Cache::tiered($l1, $l2, events: $events);
```

Veja [Observabilidade](./observabilidade.md).
