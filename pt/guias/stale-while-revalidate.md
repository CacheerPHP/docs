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
| `> stale` (>300s) | A entrada acabou; recalcula de forma síncrona (single-flight). |

Requisitos: `0 < fresh < stale`. O valor é guardado com um TTL duro de `stale`, então
o dado "stale" nunca é mais velho que a janela `stale`.

## Por que ajuda

- **Sem penhasco de latência.** Usuários na janela stale recebem resposta instantânea;
  só o refresh em segundo plano paga o custo do recálculo.
- **Sem estampede.** O refresh é coordenado por um lock, então exatamente um worker
  recarrega, mesmo sob carga.

## Refresh em segundo plano vs. inline

O refresh roda pelo **executor deferido** injetado:

- `SyncDeferredExecutor` (padrão) roda o refresh **inline**, logo após o valor stale
  ser retornado — simples, mas a requisição atual paga por ele.
- `AfterResponseDeferredExecutor` enfileira o refresh e o descarrega após a resposta
  ser enviada (no shutdown / `fastcgi_finish_request`), então o usuário não espera nem
  pela leitura stale nem pelo refresh.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Support\AfterResponseDeferredExecutor;

$cache = new Cache($store, executor: new AfterResponseDeferredExecutor());
```

O CacheerPHP nunca chama um refresh de "background" a menos que um executor deferido
que de fato defira esteja ativo.

## `flexible()` vs. `remember()`

- Use **`remember()`** quando uma resposta um pouco mais lenta na expiração for
  aceitável e você quiser o comportamento correto mais simples.
- Use **`flexible()`** para valores quentes e caros onde um usuário nunca deve esperar
  por um recálculo — ao custo de servir dados ocasionalmente até `stale` segundos
  antigos.

Veja também o [guia de Políticas](./politicas.md) para `serveStaleOnError`, que serve
dados stale especificamente quando o refresh *falha*.
