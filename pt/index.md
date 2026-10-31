# Documentação CacheerPHP (PT)

**Documentação: 6.x estável** | PHP 8.3+ | [Guia de atualização](./atualizacao/index.md)

**Cache para PHP independente de framework, com invalidação por escopo, stores componíveis e monitorização opcional.** Use a mesma interface em PHP puro, Laravel ou num worker.

Comece por executar um cache persistente duas vezes e invalide o resultado depois de alterar a origem. Precisa apenas de PHP 8.3+, Composer e um diretório com permissão de escrita fora da raiz pública.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$catalog = Cacheer::file(__DIR__ . '/var/cache')->scope('catalog');
$products = $catalog->remember('products', 300, static fn (): array => [
    ['id' => 1, 'name' => 'Desk lamp'],
]);
```

---

## Escolha uma tarefa

| Quero… | Começar aqui |
|---|---|
| Instalar o CacheerPHP | [Primeiros passos](./primeiros-passos/index.md) |
| Correr o cache do Laravel sobre o CacheerPHP | [CacheerPHP para Laravel](./laravel/index.md) |
| Usar CacheerPHP nos serviços Laravel | [Integração com Laravel](./guias/laravel.md) |
| Comparar com Laravel Cache | [Comparação e compromissos](./guias/cacheerphp-vs-laravel-cache.md) |
| Escolher o armazenamento | [Stores e capacidades](./api/drivers.md) |
| Guardar resultados dispendiosos | [Remember e locks](./guias/remember-e-locks.md) |
| Gerir pedidos simultâneos | [Stale-while-revalidate](./guias/stale-while-revalidate.md) |
| Monitorizar e depurar | [Início rápido do Cacheer Monitor](./cacheer-monitor/quick-start.md) |
| Migrar uma aplicação v5 | [Guia de migração](./atualizacao/index.md) |

Para assinaturas de métodos, consulte a [referência da API](./api/index.md). Para exemplos executáveis, consulte os [tutoriais](./tutoriais/index.md). Verifique as [versões e o suporte](./atualizacao/suporte-de-versoes.md) antes de escolher uma versão.

## Novidades da v6.0

- **Núcleo baseado em instâncias.** Um único `Cacheer` explícito e imutável por trás
  da interface `Cache`, sobre `Key`, `Scope`, `Ttl` e `CacheEntry` tipados; o tempo
  é um `Clock` injetado.
- **Um único tipo de cache.** Escopo e policy são estado do objeto, então toda
  combinação compõe: `$cache->in('billing')->withPolicy($p)->increment('hits')`.
- **Núcleo pequeno, capacidades honestas.** Uma store implementa quatro métodos; o
  resto é declarado por interface, e `$cache->supports(...)` responde com honestidade
  mesmo através de decorators.
- **Capacidades no cache.** `increment`, `decrement`, `touch`, `tag`, `flushTag`,
  `lock`, `entries` e `prune` são métodos do cache, com o escopo aplicado — sem
  precisar alcançar a store por trás dele.
- **Escopos** substituem namespaces por string por keyspaces isolados que você
  limpa separadamente — e valem também para contadores, tags e locks.
- **Decorators componíveis.** [Camadas](./guias/cache-em-camadas.md) (L1/L2),
  [resiliente](./guias/store-resiliente.md) (fallback com circuit breaker) e
  [instrumentado](./guias/observabilidade.md) (eventos tipados + métricas).
- **Proteção contra estampede.** [`remember()`](./guias/remember-e-locks.md) com
  coordenação por locks e [`flexible()`](./guias/stale-while-revalidate.md).
- **Armazenamento configurável.** Serialização, gzip opcional e criptografia autenticada AES-256-GCM opcional, com [rotação de chaves](./guias/criptografia-e-compressao.md). A autenticação exige criptografia.
- **Padrões e ferramentas.** Adaptadores [PSR-16 e PSR-6](./api/psr16-adapter.md),
  log PSR-3, ponte PSR-14 e uma [CLI `cacheer`](./guias/cli.md).
- **Migração.** Conjunto Rector opcional e uma atualização com keyspace frio — a
  renomeação é a migração, sem shim de runtime. Veja o
  [guia de atualização](./atualizacao/index.md).

## Breaking changes em resumo

- A fachada estática/global foi removida — construa e injete um `Cacheer`. Sem shim
  v5 drop-in; migre com o Rector + tabela de mapeamento, ou fique em `^5.2`.
- `get()` não recebe mais TTL de leitura; namespaces posicionais viram `scope()`
  (ou seu alias `in()`); o sucesso é o retorno ou `entry()->isHit()`, não estado
  mutável.
- Os verbos familiares da v5 continuam aqui: `forever()`, `rememberForever()`,
  `missing()`, `add()`, `pull()` (o `getAndForget` da v5),
  `increment()`/`decrement()`, `touch()` (o `renewCache` da v5), `tag()`/`flushTag()`,
  `lock()` e `stats()`.
- PHP mínimo é **8.3**. Clientes de driver e extensões são opcionais.

---

Contribuições são bem-vindas. Veja o README na raiz para estrutura e orientações.
