# CacheerPHP vs Laravel Cache

Escolha com base no ambiente da aplicação, na invalidação dos dados e no controlo de armazenamento de que precisa. Ambos oferecem uma API de cache legível. O CacheerPHP mantém essa API independente do framework; o Laravel Cache já está integrado no Laravel.

Esta comparação cobre **CacheerPHP 6.x** e **Laravel 13.x**, verificados em **20 de outubro de 2026**. Compara comportamento e integração, sem classificações de desempenho.

## Quando Laravel Cache é a escolha mais simples

Use Laravel Cache quando as suas stores e convenções já resolvem o problema. A aplicação já dispõe de configuração, bindings, eventos e ferramentas de teste. Read-through, stale-while-revalidate, locks e failover também estão disponíveis. Consulte a [documentação de cache do Laravel](https://laravel.com/docs/13.x/cache).

O [componente Illuminate Cache](https://github.com/illuminate/cache/blob/13.x/composer.json) também pode ser instalado separadamente; Laravel Cache não está limitado a aplicações Laravel completas.

## Quando considerar CacheerPHP

- **Partilha serviços entre projetos PHP.** Injete a mesma interface `Contracts\Cache` no Laravel, num serviço em PHP puro ou num worker.
- **Precisa de limites de invalidação explícitos.** Limpe o escopo de um catálogo ou tenant separadamente. Escopos qualificam chaves, tags, contadores e nomes de locks. As quatro stores integradas suportam tags e limpeza por escopo.
- **Quer compor o comportamento do armazenamento.** Configure L1/L2, fallback com circuit breaker, políticas de TTL e compressão/criptografia opcionais em instâncias explícitas.
- **Quer visibilidade sobre o CacheerPHP.** Adicione o dashboard local opcional Cacheer Monitor para inspecionar operações, tempos e atividade das chaves. A captura de valores exige ativação.

## Comparar o comportamento

| Decisão | CacheerPHP 6.x | Laravel Cache 13.x |
| --- | --- | --- |
| Integração | Construa e injete `Contracts\Cache`; no Laravel, use o driver de cache `cacheer` por trás de `Cache::` ou registe o binding | Configuração, facade e contratos integrados; componente Illuminate separado disponível |
| Armazenamento integrado | Ficheiros, array, base de dados PDO e Redis | Inclui também Memcached, DynamoDB e armazenamento em discos de filesystem |
| Guardar resultados calculados | `remember()` coordena misses quando há locks, dentro dos limites de lease/timeout | `remember()`; locks explícitos disponíveis |
| Servir dados stale | `flexible()`; executor predefinido síncrono, execução diferida configurável | `flexible()` usa o ciclo de execução diferida do Laravel |
| Tags | Suportadas nas quatro stores integradas | Sem suporte nas stores file, database, DynamoDB e storage |
| Limpar um espaço lógico | `scope('catalog')->clear()`; a limpeza da raiz respeita o espaço da store configurada | Tags conforme o driver ou stores separadas; `flush()` ignora os prefixos configurados |
| Reutilizar leituras localmente | `TieredStore` compõe L1/L2, preserva expiração e limita a idade de L1 | `memo()` mantém leituras durante um pedido ou job |
| Falha do backend | Fallback com circuit breaker para indisponibilidade; contadores e locks falham sem recorrer ao fallback | Store `failover` com lista ordenada |
| `null` em cache | Hit válido, incluindo em `remember()` e `has()` | `get()`/`remember()` tratam null como miss; `has()` devolve false |

Referências Laravel: [funcionalidades e limites dos drivers](https://laravel.com/docs/13.x/cache), [API do repository](https://api.laravel.com/docs/13.x/Illuminate/Cache/Repository.html) e [implementação de null/read-through](https://github.com/laravel/framework/blob/13.x/src/Illuminate/Cache/Repository.php). Detalhes CacheerPHP: [stores e capacidades](../api/drivers.md), [escopos](./escopos.md), [camadas](./cache-em-camadas.md) e [resiliência](./store-resiliente.md).

## A mesma tarefa

Laravel Cache:

```php
use Illuminate\Support\Facades\Cache;

$products = Cache::remember('catalog:products', 300, fn () => $repository->all());
```

CacheerPHP, com o cache fornecido pelo bootstrap ou container da aplicação:

```php
use Silviooosilva\CacheerPhp\Contracts\Cache;

function products(Cache $cache, callable $loadProducts): array
{
    return $cache->scope('catalog')->remember('products', 300, $loadProducts);
}
```

A distinção útil está na configuração, composição e invalidação. Nomes de métodos semelhantes não tornam os sistemas intercambiáveis: tenha em conta null, regras de TTL, tags e locks ao migrar. Os formatos de armazenamento também são diferentes; recalcule os valores em vez de partilhar entradas internas.

## Experimentar o caminho adequado

- [Executar um exemplo completo de CacheerPHP](../primeiros-passos/index.md): instalar, observar um miss e um hit e invalidar um resultado.
- [Correr o cache do Laravel sobre o CacheerPHP](../laravel/index.md): o driver `cacheer` mantém `Cache::`, locks, filas e scheduler, com o CacheerPHP por baixo.
- [Registar o CacheerPHP nos serviços Laravel](./laravel.md): binding num provider e um serviço de catálogo com injeção, mantendo o cache do Laravel como está.
- [Ligar o Cacheer Monitor](../cacheer-monitor/quick-start.md): inspecionar o comportamento antes de adicionar mais camadas.
- [Manter o cache nativo do Laravel](https://laravel.com/docs/13.x/cache): seguir as convenções integradas quando resolvem o problema.
