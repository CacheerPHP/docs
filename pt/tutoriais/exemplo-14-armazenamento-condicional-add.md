# Escritas condicionais e contadores atômicos

## `add()` — armazenar apenas se ausente

`add()` escreve apenas quando a chave está faltando e informa se foi *esta* chamada
que armazenou. Quando a store sabe travar, a verificação e a escrita são
serializadas, então é um "primeiro que escreve vence" correto entre processos — e
não o `has()` + `set()` sujeito a corrida que você escreveria à mão.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

if ($cache->add('import:running', 1, ttl: 300)) {
    // ganhamos — executa a importação
} else {
    // outro processo já está cuidando disso
}
```

Um valor falsy armazenado ainda é um valor, então `add()` não sobrescreve um
`false`, `0` ou `''` já armazenado.

## Contadores — `increment()` / `decrement()`

Contadores são a capacidade `AtomicStore`, atômicos até a garantia do backend, e
chamados no cache para que o escopo seja aplicado.

```php
// increment(string|Key $key, int $amount = 1, ?int $initial = null, $ttl = null): int
$cache->increment('page:views');                      // 1 (sem $initial a chave precisa existir)
$cache->increment('page:views', 5);                   // 6
$cache->increment('score', 1, initial: 100);          // começa em 100 + 1 = 101
$cache->decrement('stock', 1);                        // lê melhor que um valor negativo

// Contador com janela de tempo — rate limit.
$cache->increment('rate:user:99', 1, initial: 0, ttl: '1 minute');
```

Contadores respeitam o escopo, então tenants não colidem:

```php
$cache->in('tenant-a')->increment('signups', 1, initial: 0);  // 1
$cache->in('tenant-b')->increment('signups', 5, initial: 0);  // 5
```

## Efeitos colaterais de uma vez só — `lock()`

`add()` protege uma entrada de cache; um lock protege o seu *trabalho*. Use um lock
quando o efeito colateral precisa acontecer exatamente uma vez.

```php
$lock = $cache->lock('job:send_invoice:42', 60);
if ($lock->acquire()) {
    try {
        $invoices->send(42);
    } finally {
        $lock->release();
    }
}
```

## Compare-and-swap

Para concorrência otimista, `compareAndSwap()` escreve apenas se o valor atual ainda
for o que você leu por último. É uma primitiva do contrato da store, alcançada via
`store()`:

```php
use Silviooosilva\CacheerPhp\Kernel\Key;

// compareAndSwap(Key, mixed $expected, mixed $value, ?Ttl = null): bool
$ok = $cache->store()->compareAndSwap(Key::named('job:state'), 'idle', 'running');
```

Garantias por backend: `ArrayStore` é atômico dentro de um processo; `FileStore`
serializa com um lock de arquivo por chave; `DatabaseStore` usa read-modify-write com
lock de linha; `RedisStore` usa atômicos do servidor. Veja
[Remember e locks](../guias/remember-e-locks.md#contadores-atomicos).
