# Locks e seções críticas

Uma store que implementa `LockingStore` (as quatro nativas) entrega locks nomeados e
limitados por TTL para que só um worker rode uma seção crítica por vez.

```php
use Silviooosilva\CacheerPhp\Kernel\Ttl;
use Silviooosilva\CacheerPhp\Stores\FileStore;

$store = new FileStore(__DIR__ . '/cache');

$lock = $store->lock('nightly-report', Ttl::minutes(5));

// Espera até 10s pelo lock; pula a execução se outro o mantém.
if (! $lock->block(10.0)) {
    return;
}

try {
    generate_report();
} finally {
    $lock->release();
}
```

Variante não bloqueante:

```php
$lock = $store->lock('import', Ttl::seconds(30));

if ($lock->acquire()) {          // true só se o pegamos agora
    try {
        run_import();
    } finally {
        $lock->release();
    }
}
```

- Locks carregam um **TTL** e autoexpiram, então um dono que travou nunca causa deadlock
  no keyspace.
- A liberação é **do dono**: um lock só apaga o próprio token.
- `remember()` usa esses locks internamente para
  [proteção contra estampede](./exemplo-18-protecao-stampede-swr.md).

Veja a [referência de Locks](../api/locks.md).
