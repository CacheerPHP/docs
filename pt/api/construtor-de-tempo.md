# TTL e Clock

> A API fluente `TimeBuilder` da v5 é substituída pelo objeto de valor
> [`Ttl`](./construtor-de-opcoes.md#ttl) e por um `Clock` injetado.

## Entradas de TTL aceitas

Onde um TTL é esperado (`set`, `remember`, `setMany`, políticas) você pode passar:

```php
$cache->set('k', $v, ttl: 3600);                        // int — segundos
$cache->set('k', $v, ttl: '2 hours');                   // string legível
$cache->set('k', $v, ttl: new DateInterval('PT30M'));   // DateInterval
$cache->set('k', $v, ttl: Ttl::minutes(30));            // objeto Ttl
$cache->set('k', $v, ttl: null);                        // para sempre
$cache->set('k', $v, ttl: 'forever');                   // para sempre (string)
```

Strings legíveis batem com `<n> second|minute|hour|day|week` (singular ou plural),
além do literal `forever`. Uma string só de dígitos (`'3600'`) é tratada como
segundos. Qualquer outra coisa lança `InvalidTtlException`.

## O objeto `Ttl`

```php
use Silviooosilva\CacheerPhp\Kernel\Ttl;

Ttl::seconds(30);   Ttl::minutes(10);   Ttl::hours(2);
Ttl::days(7);       Ttl::weeks(2);      Ttl::forever();
Ttl::until(new DateTimeImmutable('2026-01-01'), $clock);
```

- `seconds()` (e seus múltiplos) exigem um valor **maior que zero** —
  `InvalidTtlException` caso contrário. Não existe regra de "TTL zero apaga"; a
  remoção é explícita.
- O overflow é protegido: uma duração ou expiração que excederia `PHP_INT_MAX` lança
  em vez de dar wrap. "Para sempre" é representado como "sem expiração", não como um
  inteiro gigante.

## O `Clock`

Toda leitura de tempo passa por um `Clock`, então o comportamento é determinístico e
testável.

```php
namespace Silviooosilva\CacheerPhp\Contracts;

interface Clock
{
    public function now(): int;          // segundos unix
    public function nowFloat(): float;   // precisão sub-segundo
    public function sleep(int $microseconds): void;
}
```

- **Produção:** `Silviooosilva\CacheerPhp\Support\SystemClock`.
- **Testes:** um clock falso que você avança à mão — sem `sleep()`.

```php
use Silviooosilva\CacheerPhp\Support\SystemClock;

$cache = Cache::file('/var/cache', clock: new SystemClock());
```

Como o clock é injetado em toda parte (stores, decorators, políticas, locks,
adaptadores PSR), você pode congelar ou adiantar o tempo em testes para atingir os
limites exatos de expiração e das janelas de stale. Veja o
[guia de stores personalizadas](../guias/stores-personalizadas.md) para como uma
store deve usar o clock em vez de `time()`.
