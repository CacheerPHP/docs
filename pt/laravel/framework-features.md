# Funcionalidades do Laravel sobre o CacheerPHP

Numa [store encaminhada](./index.md#stores-encaminhadas), as funcionalidades do Laravel que usam o cache funcionam exatamente como antes: as suas operações vão diretamente para a store. Com uma store `cacheer` como predefinida, correm sobre o armazenamento próprio do CacheerPHP sem alterações de código; esta página descreve o que esperar de cada uma.

## A API de cache

`Cache::get()`, `put()`, `many()`, `putMany()`, `remember()`, `forever()`, `forget()` e o helper `cache()` comportam-se como nas stores do Laravel. Alguns detalhes:

- `Cache::add()` é atómico entre processos: o CacheerPHP serializa a verificação e a escrita com um lock.
- `Cache::increment()` e `decrement()` começam um contador inexistente em zero. O novo contador nunca expira; um contador existente mantém o seu tempo de vida. Os valores têm de ser números inteiros.
- `Cache::touch()` prolonga o tempo de vida de uma entrada válida e nunca reativa uma entrada expirada.
- `Cache::flush()` limpa apenas o [prefixo](./configuration.md#prefixo) da store e mantém os locks em uso.
- `Cache::putMany()` aceita chaves numéricas, como modelos indexados pelo ID.
- `Cache::flexible()` usa a implementação do próprio Laravel, que guarda os seus timestamps e o lock de atualização através do driver.

## Locks

```php
use Illuminate\Support\Facades\Cache;

Cache::lock('reports:monthly', 120)->get(function () {
    // Corre num processo de cada vez.
});
```

Os locks mantêm a semântica do Laravel:

- `$lock->owner()` e `Cache::restoreLock($name, $owner)` permitem que outro processo liberte um lock, como fazem os jobs em fila.
- `forceRelease()` liberta um lock, seja qual for o dono.
- `refresh()` reinicia o tempo de vida, apenas para o dono atual. Um lock reconstruído com `restoreLock()` não conhece o seu tempo de vida, por isso `refresh()` sem segundos mantém o atual.
- Um lock sem tempo de vida (`Cache::lock('name')`) fica retido até ser libertado.

A libertação é segura quanto ao dono: se o lock de um processo expirar e outro processo o adquirir, o primeiro já não o pode libertar. Durante uma falha do backend, os locks recusam por segurança: `get()` devolve `false`. O custo são viagens de ida e volta adicionais; consulte [diferenças e limitações](./limitations.md#os-locks-custam-mais-viagens-de-ida-e-volta).

## Filas e scheduler

Estas funcionalidades do Laravel usam locks de cache e funcionam numa store `cacheer`:

- **Jobs únicos** (`ShouldBeUnique`): um dispatch duplicado é descartado enquanto o job está pendente, e o lock é libertado quando o job termina.
- **`withoutOverlapping()` do scheduler**: uma tarefa não começa enquanto a execução anterior ainda decorre.
- **Comandos isoláveis**: `php artisan seu:comando --isolated` termina se outra instância estiver a correr.

Usam a store de cache predefinida, a menos que a funcionalidade indique outra, por exemplo com `uniqueVia()` num job.

## Rate limiting

`RateLimiter::attempt()`, `hit()` e `tooManyAttempts()`, e o middleware `throttle` para limitadores baseados no cache, contam os pedidos com `add()` e `increment()` atómicos, por isso os limites mantêm-se entre workers que partilham o backend.

Enquanto o backend principal está em baixo, `add()` e `increment()` lançam exceções, por isso as rotas com limite falham mesmo com um fallback configurado. Consulte [contadores e locks durante uma falha](./limitations.md#contadores-e-locks-durante-uma-falha).

## Funcionalidades do CacheerPHP além de `Cache::`

O driver regista o contrato `Cache` do CacheerPHP com o cache por trás da store `cacheer` predefinida (ou da primeira store `cacheer`). Injete-o onde precisar de funcionalidades que `Cache::` não expressa:

```php
use Silviooosilva\CacheerPhp\Contracts\Cache;

final class ProductCatalog
{
    public function __construct(private readonly Cache $cache)
    {
    }

    public function products(): array
    {
        return $this->cache->in('catalog')->remember('products', 300, fn (): array => $this->load());
    }

    public function forgetCategory(string $category): void
    {
        $this->cache->flushTag("category:{$category}");
    }
}
```

Consulte [escopos](../guias/escopos.md), [tags](../tutoriais/exemplo-10-tagging.md) e a [API de cache](../api/funcoes-cache.md).

## Monitorização

Instale o [Cacheer Monitor](../cacheer-monitor/quick-start.md) e todas as stores encaminhadas e todas as stores `cacheer` passam a reportar as suas operações, sem mais configuração. As stores encaminhadas aparecem todas como `LaravelStore`; consulte [stores encaminhadas](./limitations.md#stores-encaminhadas).

## Testes

As entradas expiram segundo o relógio do Laravel, por isso a viagem no tempo funciona como nas stores do Laravel:

```php
Cache::put('token', 'abc', 60);

$this->travel(61)->seconds();

$this->assertNull(Cache::get('token'));
```

Para uma store que só existe durante o teste, use `'backend' => 'array'`, por exemplo no `phpunit.xml`:

```xml
<env name="CACHE_STORE" value="cacheer_testing"/>
```

com a store correspondente em `config/cache.php`:

```php
'cacheer_testing' => ['driver' => 'cacheer', 'backend' => 'array'],
```
