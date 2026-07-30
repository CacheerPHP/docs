# Stores e capacidades

Uma **store** é onde as entradas vivem. A v6 mantém o contrato obrigatório mínimo e
move tudo o que é opcional para **interfaces de capacidade** que a store declara
implementando-as. O núcleo verifica a capacidade e lança
`UnsupportedCapabilityException` se ela faltar — nunca finge uma garantia.

## O contrato `Store`

```php
namespace Silviooosilva\CacheerPhp\Contracts;

interface Store
{
    public function get(Key $key): CacheEntry;             // hit ou miss
    public function set(Key $key, mixed $value, Ttl $ttl): void;
    public function delete(Key $key): bool;               // true se removeu
    public function clear(): void;                        // apenas o keyspace desta store
}
```

## Interfaces de capacidade

| Interface | Métodos | Adiciona |
|---|---|---|
| `BatchStore` | `getMany`, `setMany`, `deleteMany` | Operações multi-chave nativas |
| `TouchStore` | `touch(Key, Ttl): bool` | Estender o TTL de uma entrada no lugar |
| `PrunableStore` | `prune(): int` | Remover entradas expiradas; retorna a contagem |
| `InspectableStore` | `entries(?Scope): iterable` | Iterar entradas vivas / metadados |
| `FlushableScopeStore` | `clearScope(Scope): void` | Limpar um único escopo |
| `TaggableStore` | `tag(Key, ...string)`, `clearTag(string): int` | Agrupar + invalidar por tag |
| `AtomicStore` | `increment(Key, int $amount = 1, ?int $initial = null, ?Ttl = null): int`, `compareAndSwap(Key, mixed $expected, mixed $value, ?Ttl = null): bool` | Contadores atômicos e CAS |
| `LockingStore` | `lock(string $name, Ttl): Lock` | Locks nomeados — veja [Locks](./locks.md) |

## Stores nativas

As quatro stores nativas implementam **todas** as capacidades acima. O que difere é
a *garantia* por trás de cada capacidade, principalmente atomicidade e locking.

| | `ArrayStore` | `FileStore` | `DatabaseStore` | `RedisStore` |
|---|:--:|:--:|:--:|:--:|
| Persistência | Em processo | Disco | PDO (SQLite/MySQL/PgSQL) | Servidor Redis |
| Dependências | nenhuma | nenhuma | `ext-pdo` | `predis` ou `ext-redis` |
| Escopo atômico | uma requisição | um host (file lock) | row lock / txn | server-side |
| Melhor para | testes, CLI curta | host único | estado compartilhado | alta vazão |

```php
use Silviooosilva\CacheerPhp\Stores\{ArrayStore, FileStore, DatabaseStore, RedisStore};

$store = new ArrayStore($clock);
$store = new FileStore('/var/cache/app', $codec, clock: $clock);
$store = new DatabaseStore($pdo, 'cacheer_store', $codec, clock: $clock);
$store = new RedisStore($connection, 'cacheer', $codec, clock: $clock);
```

O `$codec` vem de um [`PipelineConfig`](./configuracao.md); omita-o para o padrão
seguro. Prefira os [construtores nomeados](./funcoes-cache.md#construtores-nomeados)
a menos que precise da store crua.

### Conexões Redis

`RedisStore` recebe uma `RedisConnection` injetada — nunca cria um cliente global:

```php
use Silviooosilva\CacheerPhp\Stores\Support\{PredisConnection, PhpRedisConnection};

$connection = new PredisConnection($predisClient);    // predis/predis
$connection = new PhpRedisConnection($redis);         // ext-redis \Redis
```

### Schema do banco

`DatabaseStore` nunca cria o schema implicitamente. Migre uma vez, explicitamente:

```php
use Silviooosilva\CacheerPhp\Stores\Support\DatabaseStoreSchema;

DatabaseStoreSchema::migrate($pdo, 'cacheer_store');  // idempotente
DatabaseStoreSchema::drop($pdo, 'cacheer_store');     // rollback
```

Ou veja o DDL com `cacheer migrate --dry-run` (veja [CLI](../guias/cli.md)).

## Decorators

Decorators embrulham qualquer store e repassam todas as capacidades que a(s)
store(s) embrulhada(s) oferecem, então a composição nunca perde uma funcionalidade.

- **[`TieredStore`](../guias/cache-em-camadas.md)** — um L1 local rápido na frente de
  um L2 compartilhado, com promoção e coerência por geração.
- **[`ResilientStore`](../guias/store-resiliente.md)** — primário com fallback,
  protegido por um circuit breaker.
- **[`InstrumentedStore`](../guias/observabilidade.md)** — cronometra cada operação e
  emite eventos tipados; valores nunca são capturados a menos que você opte.

```php
$cache = Cacheer::tiered(new ArrayStore($clock), $redisStore);
$cache = Cacheer::resilient($primary, $fallback);
$cache = Cacheer::instrumented($store, $events);
```

## Escrevendo a sua

Implemente `Store`, adicione as capacidades que você garante e comprove com a suíte
de conformidade compartilhada. Veja [Stores personalizadas](../guias/stores-personalizadas.md).
