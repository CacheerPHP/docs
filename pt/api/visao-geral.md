# Visão geral da arquitetura

O CacheerPHP 6 é construído a partir de peças pequenas e componíveis. Você
interage com um núcleo `Cacheer`; ele delega a uma `Store`; decorators opcionais
adicionam camadas, resiliência e instrumentação; e um pipeline de armazenamento
transforma valores em um envelope seguro e versionado.

```text
API pública
  Cacheer / ScopedCacheer / PolicyCacheer
          |
Objetos de valor do núcleo
  Key / Scope / Ttl / CacheEntry / Clock
          |
Decorators (opcionais, componíveis)
  TieredStore / ResilientStore / InstrumentedStore
          |
Contrato Store (+ capacidades opcionais)
  Store: get / set / delete / clear
          |
Stores nativas
  ArrayStore / FileStore / DatabaseStore / RedisStore
          |
Pipeline de armazenamento
  serialize -> comprime -> criptografa-autenticado -> Envelope versionado
```

## Namespaces

| Namespace | Conteúdo |
|---|---|
| `Kernel\` | `Cacheer`, `ScopedCacheer`, `PolicyCacheer` e os objetos de valor `Key`, `Scope`, `Ttl`, `CacheEntry` |
| `Contracts\` | `Store` e as interfaces de capacidade (`BatchStore`, `TaggableStore`, `LockingStore`, `AtomicStore`, `TouchStore`, `PrunableStore`, `InspectableStore`, `FlushableScopeStore`), além de `Clock`, `Lock`, `DeferredExecutor`, `EventDispatcher`, `RedisConnection` |
| `Stores\` | `ArrayStore`, `FileStore`, `DatabaseStore`, `RedisStore` e os decorators `TieredStore`, `ResilientStore`, `InstrumentedStore` |
| `Storage\` | `Envelope`, `EnvelopeCodec`, serializers, compressão, criptografia, codificação de chave e o leitor v5 |
| `Config\` | `PipelineConfig`, `CachePolicy` |
| `Support\` | `SystemClock`, `CircuitBreaker`, executores deferidos |
| `Observability\` | `CacheEvent`, `CacheEventType`, `EventBus`, `MetricsCollector`, pontes PSR-3/PSR-14 |
| `Psr\` | `Psr16Cache`, `Psr6Pool`, `Psr6Item` |
| `Console\` | a CLI de operações `cacheer` |

## O contrato Store mínimo

Uma store só precisa implementar quatro métodos:

```php
interface Store
{
    public function get(Key $key): CacheEntry;
    public function set(Key $key, mixed $value, Ttl $ttl): void;
    public function delete(Key $key): bool;
    public function clear(): void;
}
```

Todo o resto — lotes, tags, locks, contadores atômicos, touch, prune, inspect,
flush por escopo — é uma **capacidade opcional** que a store declara implementando
uma interface. Veja [Stores e capacidades](./drivers.md).

## Ler vs. inspecionar

- `Cacheer::get($key, $default)` retorna o valor ou seu padrão. Simples.
- `Cacheer::entry($key)` retorna um [`CacheEntry`](./construtor-de-opcoes.md#cacheentry)
  para você distinguir um `null` armazenado de um miss e ler timestamps e TTL
  restante.

## Onde fica a configuração

Não há configuração ambiente. Um `Cacheer` é exatamente o que você constrói: uma
store (construída a partir de um [`PipelineConfig`](./configuracao.md) quando
persiste), um `Clock` opcional, um executor deferido e um dispatcher de eventos. A
biblioteca nunca lê `.env`, muda o timezone nem cria schema sozinha.
