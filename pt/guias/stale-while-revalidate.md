# Stale-while-revalidate (`flexible`)

O `flexible()` mantém dados quentes rápidos **e** frescos. Em vez de uma expiração
dura que força um recálculo lento no pior momento, ele serve dados ligeiramente
stale na hora enquanto os recarrega em segundo plano.

## As três janelas

```php
$feed = $cache->flexible('feed', fresh: 30, stale: 300, callback: fn () => build_feed());
```

Dada uma entrada criada no tempo `T`:

| Idade | Comportamento |
|---|---|
| `0 .. fresh` (0–30s) | Serve o valor cacheado diretamente. |
| `fresh .. stale` (30–300s) | Serve o valor **stale** na hora e dispara **um** refresh em segundo plano. |
| `> stale` (>300s) | Nunca é servido: recalcula de forma síncrona (single-flight). |

Requisitos: `0 < fresh < stale`. O valor é guardado com um TTL duro de `stale`, e a
idade é medida a partir da criação do valor, então um valor mais velho que a janela
`stale` nunca é servido — nem um gravado sob a mesma chave com TTL maior, nem um
promovido de outra camada de cache.

## Por que ajuda

- **Sem penhasco de latência.** Usuários na janela stale recebem resposta instantânea;
  só o refresh em segundo plano paga o custo do recálculo.
- **Sem estampede.** Uma rajada de leituras stale enfileira **um** refresh por
  chave: o lock de refresh o marca como pendente até terminar, falhar ou não poder
  ser agendado (entre processos, enquanto durar a concessão do lock). Um refresh
  enfileirado verifica o frescor antes, então não faz nada se o valor já foi
  renovado. Num store sem locks, cada leitura stale enfileira uma tarefa, mas só a
  primeira calcula.

## Refresh em segundo plano vs. inline

O refresh roda pelo **executor deferido** injetado:

- `SyncDeferredExecutor` (padrão) roda o refresh **inline**, logo após o valor stale
  ser retornado — simples, mas a requisição atual paga por ele.
- `AfterResponseDeferredExecutor` enfileira o refresh e o descarrega após a resposta
  ser enviada (no shutdown / `fastcgi_finish_request`), então o usuário não espera nem
  pela leitura stale nem pelo refresh.

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Support\AfterResponseDeferredExecutor;

$cache = new Cacheer($store, executor: new AfterResponseDeferredExecutor());
```

O CacheerPHP nunca chama um refresh de "background" a menos que um executor deferido
que de fato defira esteja ativo.

### Workers de longa duração

O `AfterResponseDeferredExecutor` descarrega no shutdown. Num worker que atende
muitas requisições ou jobs no mesmo processo (workers de fila, RoadRunner, Swoole,
modo worker do FrankenPHP), o shutdown só acontece no fim, então descarregue
explicitamente após cada unidade de trabalho:

```php
$executor = new AfterResponseDeferredExecutor();
$cache = new Cacheer($store, executor: $executor);

foreach ($jobs as $job) {
    handle($job, $cache);
    $executor->flush(); // executa agora os refreshes enfileirados deste job
}
```

## `flexible()` vs. `remember()`

- Use **`remember()`** quando uma resposta um pouco mais lenta na expiração for
  aceitável e você quiser o comportamento correto mais simples.
- Use **`flexible()`** para valores quentes e caros onde um usuário nunca deve esperar
  por um recálculo — ao custo de servir dados ocasionalmente até `stale` segundos
  antigos.

Veja também o [guia de Políticas](./politicas.md) para `serveStaleOnError`, que serve
dados stale especificamente quando o refresh *falha*.
