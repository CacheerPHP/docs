# Contadores atômicos e compare-and-swap

Contadores e escritas condicionais são uma capacidade da store (`AtomicStore`),
atômicos até a garantia do backend.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Kernel\Key;
use Silviooosilva\CacheerPhp\Stores\FileStore;

$store = new FileStore(__DIR__ . '/cache');

// increment(Key, int $amount = 1, ?int $initial = null, ?Ttl $ttl = null): int
$store->increment(Key::named('page:views'));            // 1
$store->increment(Key::named('page:views'), amount: 5); // 6
$store->increment(Key::named('score'), initial: 100);   // começa em 100 + 1 = 101

// Decremento é um incremento negativo.
$store->increment(Key::named('stock'), amount: -1);
```

Compare-and-swap grava apenas se o valor atual bater — uma forma sem lock de proteger
uma transição:

```php
// compareAndSwap(Key, mixed $expected, mixed $value, ?Ttl = null): bool
$ok = $store->compareAndSwap(Key::named('job:state'), 'idle', 'running');
if ($ok) {
    // ganhamos a corrida; rode o job
}
```

Garantias por backend: `ArrayStore` é atômico dentro de um processo; `FileStore`
serializa num file lock por chave; `DatabaseStore` usa um read-modify-write com row
lock; `RedisStore` usa atômicos server-side. Veja
[Remember e locks](../guias/remember-e-locks.md#contadores-atomicos).
