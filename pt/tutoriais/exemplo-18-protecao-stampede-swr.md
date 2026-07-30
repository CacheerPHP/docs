# Proteção contra estampede e stale-while-revalidate

Duas ferramentas para chaves quentes: `remember()` evita um dogpile; `flexible()`
mantém respostas instantâneas enquanto o dado recarrega em segundo plano.

## `remember()` single-flight

Quando muitas requisições dão miss na mesma chave ao mesmo tempo, só uma calcula; as
demais esperam e leem o resultado dela.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

$value = $cache->remember('expensive', ttl: '5 minutes', callback: function () {
    return run_expensive_query(); // roda uma vez entre workers concorrentes
});
```

Isso precisa de uma store que trave (as quatro nativas). Sem uma, `remember()` ainda
funciona mas não é à prova de estampede.

## `flexible()` — stale-while-revalidate

Serve dado fresco diretamente; depois que envelhece além de `fresh`, serve o valor
**stale** na hora e o recarrega uma vez em segundo plano; após `stale`, recalcula.

```php
$feed = $cache->flexible('home:feed', fresh: 30, stale: 300, callback: fn () => build_feed());
```

| Idade | Comportamento |
|---|---|
| 0–30s | serve o valor cacheado |
| 30–300s | serve stale na hora + um refresh em segundo plano |
| >300s | recalcula de forma síncrona |

O refresh roda pelo executor deferido. Use `AfterResponseDeferredExecutor` para o
usuário não esperar nem pela leitura nem pelo refresh:

```php
use Silviooosilva\CacheerPhp\Support\AfterResponseDeferredExecutor;

$cache = new Cacheer($store, executor: new AfterResponseDeferredExecutor());
```

Veja [Remember e locks](../guias/remember-e-locks.md) e
[Stale-while-revalidate](../guias/stale-while-revalidate.md).
