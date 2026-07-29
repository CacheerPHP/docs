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
simples (sem coordenação), ainda correto, só não à prova de estampede.

## `rememberForever`

Passe `null` como TTL para uma entrada que nunca expira:

```php
$config = $cache->remember('app:config', ttl: null, callback: fn () => load_config());
```

## Usando locks diretamente

Para seções críticas que não são só cache, adquira um lock você mesmo:

```php
use Silviooosilva\CacheerPhp\Kernel\Ttl;

$lock = $store->lock('nightly-report', Ttl::minutes(5));

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
do dono. Veja a [referência de Locks](../api/locks.md).

## Contadores atômicos

Contadores são uma capacidade da store
([`AtomicStore`](../api/drivers.md#interfaces-de-capacidade)), atômicos até a garantia
do backend:

```php
use Silviooosilva\CacheerPhp\Kernel\Key;

$next = $store->increment(Key::named('page:views'));           // 1, 2, 3, ...
$store->increment(Key::named('page:views'), amount: 5);
$ok = $store->compareAndSwap(Key::named('state'), 'idle', 'running');
```

Veja [Contadores atômicos e CAS](../tutoriais/exemplo-14-armazenamento-condicional-add.md).
