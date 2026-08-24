# Referência da API

Referência completa da API pública do CacheerPHP 6.x. A v6 é baseada em
instâncias: um núcleo `Cacheer` enxuto sobre um contrato `Store` mínimo, com
capacidades opcionais e decorators componíveis.

## Núcleo

| Página | Descrição |
|--------|-----------|
| [Visão geral](./visao-geral.md) | Arquitetura, namespaces e como as peças se encaixam |
| [Cacheer](./funcoes-cache.md) | A superfície inteira: `get`/`set`, `remember`, `flexible`, capacidades, `scope`, `withPolicy` e construtores nomeados |
| [Objetos de valor](./construtor-de-opcoes.md) | `Key`, `Scope`, `Ttl`, `CacheEntry` |
| [TTL e Clock](./construtor-de-tempo.md) | Entradas de TTL, "para sempre" e o `Clock` injetado |

## Stores

| Página | Descrição |
|--------|-----------|
| [Stores e capacidades](./drivers.md) | O contrato `Store`, interfaces de capacidade, stores nativas e decorators |
| [Configuração](./configuracao.md) | Construtores nomeados e o pipeline `PipelineConfig` |
| [Compressão e criptografia](./compressao-criptografia.md) | O envelope, gzip e AES-256-GCM autenticado |
| [Locks](./locks.md) | `LockingStore`, `Lock` e coordenação single-flight |

## Interoperabilidade

| Página | Descrição |
|--------|-----------|
| [Adaptadores PSR-16 e PSR-6](./psr16-adapter.md) | `Psr16Cache`, `Psr6Pool`, `Psr6Item` |

## Convenções

- **Baseado em instâncias.** Construa um `Cacheer` e passe-o onde precisar — não há
  fachada estática nem singleton ambiente. Use a interface `Contracts\Cache` em type
  hints para que um cache com escopo ou com policy seja substituível. O único estado
  global do processo é o tap opcional
  [`Telemetry`](../guias/observabilidade.md#o-tap-global-de-telemetria).
- **Um único tipo de cache.** `scope()`, `in()` e `withPolicy()` retornam outro
  `Cacheer`, então toda combinação compõe e nada se perde pelo caminho.
- **Chaves são strings ou objetos `Key`.** Todo método que recebe uma chave aceita
  ambos; uma string simples é embrulhada como `Key::named($string)`.
- **O tempo é injetado.** Todo construtor aceita um `Clock` opcional; o padrão é
  `SystemClock`. Testes usam `FakeClock`.
- **Capacidades vivem no cache.** `increment`, `decrement`, `touch`, `tag`,
  `flushTag`, `lock`, `entries` e `prune` são métodos do `Cacheer`, com o escopo
  deste cache aplicado. Pergunte com `$cache->supports(...)` — nunca `instanceof` —
  e chamar uma que a store não honra lança `UnsupportedCapabilityException`.
- **Valores fazem round-trip sem perdas**, incluindo `null`, `false`, `0`, `''` e
  `[]`. Um `null` cacheado é um hit, não um miss — use `Cacheer::entry()` para
  distingui-los.

Veja também o [guia de atualização](../atualizacao/index.md) e os exemplos
testados em CI no diretório `examples/v6` do pacote.
