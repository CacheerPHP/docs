# Exemplo 18 — Proteção contra Stampede & Stale-While-Revalidate

*Novo na v5.2.0*

Quando uma chave popular expira, todos os pedidos em curso falham ao mesmo tempo e executam juntos o callback caro — um *cache stampede* (ou "dogpile"). O CacheerPHP agora evita isso e adiciona um modo stale-while-revalidate para que as leituras raramente bloqueiem à espera do recálculo.

## `remember()` à Prova de Stampede

`remember()` chama-se da mesma forma — a proteção é automática. Num miss, um pedido calcula sob um lock single-flight enquanto os outros esperam e depois leem o valor acabado de guardar.

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useFileDriver();

// Mesmo que 100 pedidos cheguem em simultâneo com a chave fria,
// computeExpensiveStats() corre exatamente uma vez.
$stats = $cache->remember('dashboard:stats', 300, function () {
    return computeExpensiveStats(); // query lenta, chamada a API, etc.
});
```

Funciona em qualquer driver com locking (File, Database, Redis). Também se aplica ao contexto fluente de namespace:

```php
$report = $cache->in('reports')->remember('latest', 600, fn () => buildReport());
```

## Stale-While-Revalidate com `flexible()`

`flexible()` continua a servir um valor em cache para lá do horizonte *fresh* enquanto um único worker o atualiza em background, e só bloqueia quando o valor ultrapassa o horizonte *stale*.

```php
// Fresco durante 60s; serve-stale-e-atualiza durante até 10 minutos.
$html = $cache->flexible('home', fresh: 60, stale: 600, function () {
    return renderHomepage();
});
```

Ciclo de vida de uma chave:

| Idade do valor | Comportamento |
|----------------|---------------|
| `< 60s` (fresco) | Servido diretamente. |
| `60s – 600s` (stale) | Servido de imediato; um pedido recalcula em background. |
| `> 600s` (expirado) | Recalculado sob um lock single-flight. |

O pedido que ganha o lock de atualização recalcula inline e devolve o valor fresco; todos os outros na janela stale recebem o valor em cache instantaneamente. O callback nunca corre mais do que uma vez ao mesmo tempo.

## Como escolher entre os dois

| Use | Quando |
|-----|--------|
| `remember()` | Quer sempre um valor não mais antigo do que o TTL e tolera que um pedido bloqueie num miss. |
| `flexible()` | Prefere servir um valor ligeiramente desatualizado a alguma vez bloquear leitores — dashboards, páginas iniciais, agregações caras. |

## Ver também

- [Funções de Cache → remember()](../api/funcoes-cache.md)
- [Bloqueios Distribuídos](../api/locks.md)
