# Remember, single-flight e locks

O padrão de cache mais comum é "retorne se cacheado, senão calcule, guarde e
retorne". O `remember()` faz exatamente isso — e protege do **estampede** (dogpile)
que ocorre quando muitas requisições dão miss na mesma chave ao mesmo tempo.

## `remember()`

```php
$user = $cache->remember('user:42', ttl: '10 minutes', callback: fn () => $users->find(42));
```

- Num **hit**, o callback nunca roda.
- Num **miss**, o callback roda uma vez, o resultado é guardado sob o TTL e retornado.
- Um `null`/`false`/`0` armazenado conta como hit, então `remember()` não recalcula um
  resultado legitimamente vazio a cada chamada.

## Single-flight (proteção contra estampede)

Quando a store implementa [`LockingStore`](../api/locks.md) — as quatro nativas
implementam — `remember()` é **single-flight**:

1. Muitos workers dão miss na mesma chave ao mesmo tempo.
2. Um adquire um lock curto e roda o callback.
3. Os outros bloqueiam brevemente, depois leem o valor que o primeiro acabou de
   gravar.

Então um cálculo caro roda **uma vez**, não uma por requisição concorrente. Se o lock
não puder ser adquirido na janela interna, um worker cai para calcular em vez de
bloquear indefinidamente — correção acima de coordenação.

Em uma store que não pode travar, `remember()` degrada para um compute-and-store
simples (sem coordenação), ainda correto, só não à prova de estampede. Isso também
vale através de decorators: embrulhar uma store em `TieredStore`, `ResilientStore`
ou `InstrumentedStore` nunca transforma um `remember()` funcional em falha, porque o
kernel pergunta se o travamento está *de fato* disponível em vez de confiar em
`instanceof`.

## `rememberForever()`

Para uma entrada que nunca expira:

```php
$config = $cache->rememberForever('app:config', fn () => load_config());
$config = $cache->remember('app:config', null, fn () => load_config()); // o mesmo
```

## Usando locks diretamente

Para seções críticas que não são só cache, adquira um lock você mesmo:

```php
$lock = $cache->lock('nightly-report', '5 minutes');

if (! $lock->block(10.0)) {
    return; // outro worker está executando
}

try {
    generate_report();
} finally {
    $lock->release();
}
```

Locks carregam um TTL e autoexpiram (sem deadlock se um dono travar), e a liberação é
do dono. Nomes de lock têm namespace por escopo, então
`$cache->in('tenant-a')->lock('import')` e `$cache->in('tenant-b')->lock('import')`
não disputam. Veja a [referência de Locks](../api/locks.md).

## Contadores atômicos

Contadores são a capacidade
[`AtomicStore`](../api/drivers.md#interfaces-de-capacidade) da store, chamados no
cache para que o escopo seja aplicado, e atômicos até a garantia do backend:

```php
$next = $cache->increment('page:views');                   // 1, 2, 3, ...
$cache->increment('page:views', 5);
$cache->decrement('stock', 1);
$cache->increment('rate:user:99', 1, initial: 0, ttl: '1 minute');
```

Para concorrência otimista, `compareAndSwap()` é uma primitiva do contrato da store:

```php
use Silviooosilva\CacheerPhp\Kernel\Key;

$ok = $cache->store()->compareAndSwap(Key::named('state'), 'idle', 'running');
```

Veja [Escritas condicionais e contadores atômicos](../tutoriais/exemplo-14-armazenamento-condicional-add.md).
