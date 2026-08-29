# Cacheer — referência de métodos

`Silviooosilva\CacheerPhp\Cacheer` é o ponto de entrada público, e
`Silviooosilva\CacheerPhp\Contracts\Cache` é a interface que ele implementa — a que
o código da aplicação deve usar em type hints.

Existe **um único tipo de cache**. `scope()`, `in()` e `withPolicy()` retornam
outro `Cacheer`, então nada se perde pelo caminho: um cache com escopo continua
aceitando uma policy, um cache com policy continua aceitando escopo, e ambos
continuam sabendo incrementar, taguear e travar. Todo argumento de chave aceita uma
`string` ou uma [`Key`](./construtor-de-opcoes.md#key).

```php
use Silviooosilva\CacheerPhp\Contracts\Cache;

function gerarRelatorio(Cache $cache): string { /* ... */ }

gerarRelatorio($cache);                                   // simples
gerarRelatorio($cache->in('reports')->withPolicy($p));    // com escopo + policy
```

## Construtores nomeados

```php
use Silviooosilva\CacheerPhp\Cacheer;

Cacheer::inMemory(?Clock $clock = null): Cacheer
Cacheer::file(string $directory, ?PipelineConfig $pipeline = null, ?Clock $clock = null): Cacheer
Cacheer::database(PDO $pdo, string $table = 'cacheer_store', ?PipelineConfig $pipeline = null, ?Clock $clock = null): Cacheer
Cacheer::redis(RedisConnection $connection, string $prefix = 'cacheer', ?PipelineConfig $pipeline = null, ?Clock $clock = null): Cacheer

// Decorators
Cacheer::tiered(Store $l1, Store $l2, ?Ttl $l1MaxTtl = null, ?Clock $clock = null, ?DeferredExecutor $executor = null, ?EventDispatcher $events = null): Cacheer
Cacheer::resilient(Store $primary, Store $fallback, ?CircuitBreaker $breaker = null, ?Clock $clock = null, ?DeferredExecutor $executor = null): Cacheer
Cacheer::instrumented(Store $store, EventDispatcher $events, bool $captureValues = false, ?callable $redactor = null, ?Clock $clock = null): Cacheer
```

Ou construa diretamente com qualquer store:

```php
$cache = new Cacheer($store, ?Clock $clock, ?DeferredExecutor $executor, ?EventDispatcher $events);
```

## Leitura

### `get()`

```php
public function get(string|Key $key, mixed $default = null): mixed
```

Retorna o valor cacheado, ou `$default` no miss. Um `null`/`false`/`0`/`''`
armazenado é um hit e é retornado como está — passe um sentinela como padrão, ou
use `entry()`, se precisar distinguir um `null` armazenado de um miss.

```php
$user = $cache->get('user:42');
$flag = $cache->get('feature:x', default: false);
```

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
    $ttl   = $entry->remainingTtl($clock); // segundos, ou null se for para sempre
}
```

### `has()` / `missing()`

```php
public function has(string|Key $key): bool
public function missing(string|Key $key): bool
```

`has()` é `true` quando existe uma entrada viva (não expirada); `missing()` é o
inverso, e lê melhor em uma cláusula de guarda.

```php
if ($cache->missing('user:42')) {
    return $this->rebuild();
}
```

### `many()`

```php
public function many(iterable $keys, mixed $default = null): array
```

Retorna `[chave => valor]`, usando `$default` para os misses. Usa a leitura em lote
nativa da store quando ela implementa [`BatchStore`](./drivers.md#interfaces-de-capacidade).

## Escrita

### `set()`

```php
public function set(string|Key $key, mixed $value, Ttl|DateInterval|int|string|null $ttl = null): void
```

Armazena um valor. O TTL aceita um [`Ttl`](./construtor-de-tempo.md), um `int`
(segundos), uma string legível (`'10 minutes'`), um `DateInterval` ou `null` (para
sempre).

```php
$cache->set('user:42', $user, ttl: '10 minutes');
```

### `forever()`

```php
public function forever(string|Key $key, mixed $value): void
```

Armazena sem expiração. Equivalente a `set($key, $value, null)`, mas explícito.

```php
$cache->forever('app:config', $config);
```

### `add()`

```php
public function add(string|Key $key, mixed $value, Ttl|DateInterval|int|string|null $ttl = null): bool
```

Armazena **apenas se a chave estiver ausente**, retornando `true` quando foi esta
chamada que armazenou. Quando a store sabe travar, a verificação e a escrita são
serializadas, então é um "primeiro que escreve vence" correto entre processos; caso
contrário degrada para uma verificação de processo único.

```php
if ($cache->add('import:running', 1, ttl: 300)) {
    // ganhamos — executa a importação
}
```

Um valor falsy armazenado ainda é um valor: `add()` não sobrescreve um `false`, `0`
ou `''` já armazenado.

### `setMany()`

```php
public function setMany(iterable $values, Ttl|DateInterval|int|string|null $ttl = null): void
```

Armazena `['chave' => valor, ...]` sob um único TTL. Usa lote/transação nativa
quando a store implementa `BatchStore`. As chaves precisam ser strings.

### `delete()` / `deleteMany()`

```php
public function delete(string|Key $key): bool
public function deleteMany(iterable $keys): bool
```

`delete()` retorna `true` quando algo foi removido. `deleteMany()` retorna `true`
somente se todas as chaves foram removidas.

### `pull()`

```php
public function pull(string|Key $key, mixed $default = null): mixed
```

Lê e apaga em uma única chamada, retornando o valor que a chave guardava (ou
`$default` no miss). Útil para valores de uso único — mensagens flash, tokens
descartáveis, itens de trabalho reivindicados.

```php
$message = $cache->pull('flash:user:42');
```

Como `get()`, reporta um `null` armazenado como o valor que ele é, não como miss.

### `clear()`

```php
public function clear(): void
```

Remove tudo no keyspace deste cache. Em um cache **com escopo**, remove apenas
aquele escopo (requer [`FlushableScopeStore`](./drivers.md#interfaces-de-capacidade)).

## Computar e armazenar

### `remember()`

```php
public function remember(string|Key $key, Ttl|DateInterval|int|string|null $ttl, callable $callback): mixed
```

Retorna o valor cacheado; no miss, executa `$callback`, armazena o resultado sob o
`$ttl` e o retorna. Quando a store sabe travar, `remember()` é **single-flight**:
um chamador computa enquanto os demais aguardam e leem o resultado (sem dogpile).
Sem travamento, degrada para um simples computar-e-armazenar — nunca falha por
falta de lock, inclusive quando a store está embrulhada em um decorator.

```php
$user = $cache->remember('user:42', '10 minutes', fn () => $users->find(42));
```

### `rememberForever()`

```php
public function rememberForever(string|Key $key, callable $callback): mixed
```

`remember()` sem expiração.

### `flexible()` — stale-while-revalidate

```php
public function flexible(string|Key $key, int $fresh, int $stale, callable $callback): mixed
```

Serve o valor diretamente enquanto ele está *fresco* (dentro de `$fresh` segundos
da criação). Entre `$fresh` e `$stale` serve o valor **obsoleto** imediatamente e o
atualiza uma vez, em segundo plano, via executor diferido. Passado `$stale`,
recomputa de forma síncrona. Requer `0 < fresh < stale`.

```php
$feed = $cache->flexible('feed', fresh: 30, stale: 300, callback: fn () => build_feed());
```

A janela `$stale` é um contrato explícito seu, então uma
[policy](../guias/politicas.md) vinculada nunca a reformata com jitter ou cache
negativo.

## Capacidades

As capacidades são implementadas pela **store** e chamadas no **cache**, então o
escopo deste cache é aplicado para você e uma única exceção clara aponta o que o
backend não sabe fazer. As quatro stores nativas suportam todas elas.

### `supports()`

```php
public function supports(string $capability): bool
```

Responde com honestidade mesmo através de decorators — pergunte antes de chamar se
o seu backend é plugável. Nunca use `instanceof` em uma store para isso; veja
[Stores e capacidades](./drivers.md).

```php
use Silviooosilva\CacheerPhp\Contracts\AtomicStore;

if ($cache->supports(AtomicStore::class)) {
    $cache->increment('visits');
}
```

### `increment()` / `decrement()`

```php
public function increment(string|Key $key, int $amount = 1, ?int $initial = null, Ttl|DateInterval|int|string|null $ttl = null): int
public function decrement(string|Key $key, int $amount = 1, ?int $initial = null, Ttl|DateInterval|int|string|null $ttl = null): int
```

Ajustam um contador atomicamente e retornam o novo valor. Por padrão, uma chave
ausente começa em zero; quando fornecido, `$initial` substitui esse valor inicial.
Depois, `$amount` é aplicado. O `$ttl` opcional é usado quando o contador é criado.
Requer `AtomicStore`.

```php
$cache->increment('page-views', 1, initial: 0);
$cache->decrement('stock:sku-1', 5, initial: 100);
$cache->increment('rate:user:99', 1, initial: 0, ttl: '1 minute');
```

### `touch()`

```php
public function touch(string|Key $key, Ttl|DateInterval|int|string $ttl): bool
```

Estende a vida de uma entrada **sem reescrever seu valor**. `false` quando a chave
está ausente. Requer `TouchStore`. (É o `renewCache()` da v5.)

```php
$cache->touch('session:abc', '1 hour');
```

### `tag()` / `flushTag()`

```php
public function tag(string|Key $key, string ...$tags): void
public function flushTag(string $tag): int
```

Associam uma chave a tags e depois as invalidam em bloco; `flushTag()` retorna
quantas entradas foram removidas. Os nomes de tag têm namespace por escopo, então
dois escopos usando o mesmo nome não limpam um ao outro. Requer `TaggableStore`.

```php
$cache->tag('product:1', 'products', 'catalog');
$removed = $cache->flushTag('products');
```

### `lock()`

```php
public function lock(string $name, Ttl|DateInterval|int|string $ttl = 60): Lock
```

Um mutex nomeado entre processos, com namespace por escopo. Requer `LockingStore`.
Veja [Locks](./locks.md).

```php
$lock = $cache->lock('nightly-import', 300);
if ($lock->acquire()) {
    try { /* ... */ } finally { $lock->release(); }
}
```

### `entries()` / `prune()`

```php
public function entries(): iterable   // requer InspectableStore
public function prune(): int          // requer PrunableStore
```

`entries()` percorre as entradas vivas no escopo deste cache, com metadados.
`prune()` descarta entradas expiradas e retorna quantas foram removidas.

## Escopos, policies e visões

### `scope()` / `in()`

```php
public function scope(string|Scope $scope): static
public function in(string|Scope $scope): static      // alias
public function boundScope(): Scope
```

Retornam outro `Cacheer` ligado a um keyspace isolado. Escopos aninham e se aplicam
a **todas** as operações — contadores, tags e locks inclusive. Veja
[Escopos](../guias/escopos.md).

```php
$reports = $cache->in('reports');
$reports->set('daily', $rows);
$reports->increment('runs');   // não pode colidir com outro escopo
$reports->clear();             // limpa apenas o escopo "reports"
```

### `withPolicy()`

```php
public function withPolicy(CachePolicy $policy): static
```

Vincula uma [`CachePolicy`](../guias/politicas.md) (TTL padrão, jitter, cache
negativo, serve-stale-on-error). Leituras passam direto; escritas e `remember()`
respeitam a policy. A ordem não importa — `in('x')->withPolicy($p)` e
`withPolicy($p)->in('x')` são equivalentes.

### `formatted()`

```php
public function formatted(): FormattedCacheer
```

Uma visão de formatação de leitura: leituras que retornam valor devolvem um
`CacheDataFormatter` no qual você pode encadear `->toJson()` / `->toArray()` /
`->toObject()` / `->toString()`. Todo o resto é repassado sem alteração, então a
visão mantém a superfície inteira. Use `raw()` para voltar.

```php
$json = $cache->formatted()->get('user:1')->toJson();
```

### `stats()`

```php
public function stats(): array
```

O que este cache *é* — store, escopo, se há uma policy vinculada e quais capacidades
estão realmente disponíveis. Seguro para logar ou expor; nunca contém valores
cacheados.

```php
[
    'store'        => 'FileStore',
    'scope'        => 'reports',
    'policy'       => true,
    'capabilities' => ['batch' => true, 'atomic' => true, 'locking' => true, /* ... */],
]
```

### `store()`

```php
public function store(): Store
```

A store subjacente. O código da aplicação não deve precisar dela — toda capacidade
está no cache, com o escopo aplicado. Existe para autores de stores, testes e a CLI.
