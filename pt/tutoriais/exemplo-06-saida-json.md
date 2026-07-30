# Formatando valores cacheados (JSON, array, objeto, string)

A v6 guarda valores **sem perdas** — `get()` devolve exatamente o que você colocou.
Quando quiser remodelar um valor na saída, use o `CacheDataFormatter`. Há duas
formas de acessá-lo; escolha a que ler melhor.

## 1. Embrulhe qualquer valor

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Support\CacheDataFormatter;

$cache = Cacheer::inMemory();
$cache->set('user:1', ['id' => 1, 'name' => 'Ada']);

$fmt = new CacheDataFormatter($cache->get('user:1'));

$fmt->toJson();    // '{ "id": 1, "name": "Ada" }' (bonito, UTF-8, JSON_THROW_ON_ERROR)
$fmt->toArray();   // ['id' => 1, 'name' => 'Ada']
$fmt->toObject();  // stdClass { id: 1, name: 'Ada' }
$fmt->toString();  // cast para string (melhor para escalares)
$fmt->value();     // o valor cru, sem formatação
```

## 2. A forma fluente — uma view formatada

`->formatted()` retorna uma view de leitura formatada onde `get()` já retorna um
`CacheDataFormatter`, então você encadeia em uma linha:

```php
$cache = Cacheer::file(__DIR__ . '/cache')->formatted();

$json = $cache->get('user:1')->toJson();
$arr  = $cache->get('user:1')->toArray();
```

A view também formata o `remember()`, repassa `set`/`has`/`delete`/`scope`, e
`raw()` devolve o cache subjacente:

```php
$json = $cache->remember('report', 60, fn () => build_report())->toJson();

$cache->set('k', 'v');          // escritas passam direto
$raw = $cache->raw()->get('k'); // 'v' — o valor sem formatação
```

## Por que o próprio `get()` não é o formatador

O `Cacheer::get()` base precisa retornar valores crus — um `false`, `null` ou `0`
cacheado tem que voltar como está, e os adaptadores PSR-16/PSR-6 dependem disso.
Então a formatação é **opt-in** via o wrapper standalone ou a view `formatted()`,
nunca um modo escondido no `get()`.

> `toJson()` lança `\JsonException` para um valor não codificável, em vez de
> retornar `false` silenciosamente. `toString()` segue as regras de cast do PHP —
> ideal para escalares.
