# Cache e ScopedCache — referência de métodos

`Silviooosilva\CacheerPhp\Kernel\Cache` é o ponto de entrada público. `ScopedCache`
(retornado por `scope()`) expõe a mesma superfície de leitura/escrita ligada a um
escopo. Todo argumento de chave aceita uma `string` ou uma
[`Key`](./construtor-de-opcoes.md#key).

## Construtores nomeados

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;

Cache::inMemory(?Clock $clock = null): Cache
Cache::file(string $directory, ?PipelineConfig $pipeline = null, ?Clock $clock = null): Cache
Cache::database(PDO $pdo, string $table = 'cacheer_store', ?PipelineConfig $pipeline = null, ?Clock $clock = null): Cache
Cache::redis(RedisConnection $connection, string $prefix = 'cacheer', ?PipelineConfig $pipeline = null, ?Clock $clock = null): Cache

// Decorators
Cache::tiered(Store $l1, Store $l2, ?Ttl $l1MaxTtl = null, ?Clock $clock = null, ?DeferredExecutor $executor = null, ?EventDispatcher $events = null): Cache
Cache::resilient(Store $primary, Store $fallback, ?CircuitBreaker $breaker = null, ?Clock $clock = null, ?DeferredExecutor $executor = null): Cache
Cache::instrumented(Store $store, EventDispatcher $events, bool $captureValues = false, ?callable $redactor = null, ?Clock $clock = null): Cache
```

Ou construa diretamente com qualquer store:

```php
$cache = new Cache($store, ?Clock $clock, ?DeferredExecutor $executor, ?EventDispatcher $events);
```

## Leitura

### `get()`

```php
public function get(string|Key $key, mixed $default = null): mixed
```

Retorna o valor cacheado, ou `$default` no miss. Um `null`/`false`/`0`/`''`
armazenado é um hit e é retornado como está — passe um sentinela como padrão, ou
use `entry()`, se precisar distinguir um `null` armazenado de um miss.

### `entry()`

```php
public function entry(string|Key $key): CacheEntry
```

Retorna o [`CacheEntry`](./construtor-de-opcoes.md#cacheentry) completo — hit/miss,
valor, timestamps e TTL restante.

```php
$entry = $cache->entry('user:42');
if ($entry->isHit()) {
    $value = $entry->value();
    $ttl   = $entry->remainingTtl($clock); // segundos, ou null se para sempre
}
```

### `has()` e `many()`

```php
public function has(string|Key $key): bool
public function many(iterable $keys, mixed $default = null): array
```

`has()` é `true` quando existe uma entrada viva. `many()` retorna `[chave =>
valor]`, usando `$default` para misses, com leitura em lote nativa quando a store é
[`BatchStore`](./drivers.md#interfaces-de-capacidade).

## Escrita

### `set()` e `setMany()`

```php
public function set(string|Key $key, mixed $value, Ttl|DateInterval|int|string|null $ttl = null): void
public function setMany(iterable $values, Ttl|DateInterval|int|string|null $ttl = null): void
```

O TTL aceita um [`Ttl`](./construtor-de-tempo.md), int (segundos), string legível
(`'10 minutes'`), `DateInterval` ou `null` (para sempre).

```php
$cache->set('user:42', $user, ttl: '10 minutes');
$cache->set('config', $config, ttl: null); // para sempre
```

### `delete()`, `deleteMany()`, `clear()`

```php
public function delete(string|Key $key): bool        // true se removeu
public function deleteMany(iterable $keys): bool      // true só se removeu todas
public function clear(): void                         // apenas o keyspace deste cache
```

Em um `ScopedCache`, `clear()` remove apenas aquele escopo (requer
[`FlushableScopeStore`](./drivers.md#interfaces-de-capacidade)).

## Cálculo e armazenamento

### `remember()`

```php
public function remember(string|Key $key, Ttl|DateInterval|int|string|null $ttl, callable $callback): mixed
```

Retorna o valor cacheado; no miss, executa `$callback`, guarda o resultado sob
`$ttl` e o retorna. Quando a store é [`LockingStore`](./locks.md), `remember()` é
**single-flight**: um chamador calcula enquanto os outros esperam e leem o
resultado (sem dogpile).

```php
$user = $cache->remember('user:42', '10 minutes', fn () => $users->find(42));
```

### `flexible()` — stale-while-revalidate

```php
public function flexible(string|Key $key, int $fresh, int $stale, callable $callback): mixed
```

Serve o valor diretamente enquanto está *fresco* (até `$fresh` segundos). Entre
`$fresh` e `$stale` serve o valor **stale** na hora e recarrega uma vez, em segundo
plano, via executor deferido. Após `$stale`, recalcula de forma síncrona. Requer
`0 < fresh < stale`. Veja [Stale-while-revalidate](../guias/stale-while-revalidate.md).

## Escopos e políticas

### `scope()` e `withPolicy()`

```php
public function scope(string|Scope $scope): ScopedCache
public function withPolicy(CachePolicy $policy): PolicyCache
```

`scope()` retorna um [`ScopedCache`](../guias/escopos.md) — um keyspace isolado com
a mesma API. `withPolicy()` aplica uma [`CachePolicy`](../guias/politicas.md) (TTL
padrão, jitter, cache negativo, serve-stale-on-error).

```php
$cache->scope('reports')->set('daily', $rows);
$cache = $cache->withPolicy(CachePolicy::defaults()->withTtl('10 minutes')->withJitter(0.10));
```

## Capacidades além do núcleo

Contadores atômicos, tags, touch, prune e inspeção ficam nas interfaces de
capacidade da **store**, não em `Cache`. Acesse pela store ou pela ponte
[`LegacyCacheer`](../atualizacao/index.md):

```php
$store->increment(Key::named('visits'));        // AtomicStore
$store->tag(Key::named('p1'), 'products');       // TaggableStore
$store->prune();                                 // PrunableStore
```

Veja [Stores e capacidades](./drivers.md) para a lista completa.
