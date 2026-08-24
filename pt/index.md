# Documentação CacheerPHP (PT)

**Versão atual: 6.0** | PHP 8.3+ | [Guia de atualização](./atualizacao/index.md)

O CacheerPHP 6 é um cache **baseado em instâncias**: um núcleo `Cacheer` enxuto
sobre um contrato `Store` mínimo de quatro métodos, com tudo o mais — lotes, tags,
locks, contadores atômicos, camadas, resiliência, criptografia — como uma
**capacidade opcional** que você habilita. Não há estado global nem efeito
colateral no autoload. A v5 segue na própria linha `5.x` durante a migração.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

$cache->set('user:42', $user, ttl: '10 minutes');
$user = $cache->get('user:42');

$user = $cache->remember('user:42', '10 minutes', fn () => $users->find(42));
```

---

## Seções

| Seção | Descrição |
|-------|-----------|
| [Primeiros Passos](./primeiros-passos/index.md) | Instalação, início rápido, stores, escopos, TTL, PSR |
| [Guias](./guias/configuracao.md) | Aprofundamentos: escopos, remember/locks, SWR, camadas, resiliência, políticas, criptografia, observabilidade, CLI, stores |
| [Referência da API](./api/index.md) | Referência precisa de cada classe e método público |
| [Tutoriais](./tutoriais/index.md) | Exemplos curtos e objetivos da v6 |
| [Guia de Atualização](./atualizacao/index.md) | Atualizando da v5 |
| [Guia de Contribuição](./contribuicao/index.md) | Setup, testes, conformidade, RFCs |

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
- **Proteção contra estampede.** [`remember()`](./guias/remember-e-locks.md) de
  execução única e [`flexible()`](./guias/stale-while-revalidate.md).
- **Armazenamento autenticado.** serialize → gzip opcional → AES-256-GCM opcional
  em um envelope versionado e à prova de adulteração, com
  [rotação de chaves](./guias/criptografia-e-compressao.md).
- **Padrões e ferramentas.** Adaptadores [PSR-16 e PSR-6](./api/psr16-adapter.md),
  log PSR-3, ponte PSR-14 e uma [CLI `cacheer`](./guias/cli.md).
- **Migração.** Conjunto Rector opcional mais reescrita na leitura de payloads v5
  — a renomeação é a migração, sem shim de runtime. Veja o
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
