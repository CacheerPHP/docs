# Locks

Uma store que implementa `LockingStore` pode entregar locks nomeados e limitados por
TTL. O núcleo os usa internamente para [`remember()`](./funcoes-cache.md#remember)
single-flight e stale refresh; você também pode usá-los diretamente para qualquer
seção crítica.

## `LockingStore`

```php
namespace Silviooosilva\CacheerPhp\Contracts;

interface LockingStore
{
    public function lock(string $name, Ttl $ttl): Lock;
}
```

As quatro stores nativas implementam. O *escopo* da garantia difere: `ArrayStore` é
só em processo; `FileStore` é seguro entre processos em um host (`flock`);
`DatabaseStore` usa uma linha de lock; `RedisStore` usa `SET NX` com liberação por
compare-and-delete (Lua).

## `Lock`

```php
namespace Silviooosilva\CacheerPhp\Contracts;

interface Lock
{
    public function acquire(): bool;              // não bloqueante; true se adquiriu
    public function block(float $seconds): bool;  // espera até N segundos
    public function release(): bool;              // libera só se ainda somos o dono
}
```

- Um lock carrega um **TTL** e autoexpira, então um dono que travou nunca causa
  deadlock no keyspace.
- A liberação é **do dono** (compare-and-delete): um lock só apaga o próprio token.

## Usando um lock

```php
use Silviooosilva\CacheerPhp\Kernel\Ttl;

$lock = $store->lock('import:catalog', Ttl::seconds(30));

if ($lock->acquire()) {
    try {
        run_import();
    } finally {
        $lock->release();
    }
}
```

Bloqueando com timeout:

```php
$lock = $store->lock('nightly-job', Ttl::minutes(5));

if (! $lock->block(10.0)) {
    return; // outro worker o mantém; pule esta execução
}

try {
    run_job();
} finally {
    $lock->release();
}
```

## `remember()` single-flight

Quando a store é `LockingStore`, `remember()` fica automaticamente à prova de
estampede: num miss concorrente, um chamador adquire o lock e calcula enquanto os
outros bloqueiam brevemente e leem o valor recém-gravado. Se o lock não puder ser
adquirido na janela interna, os chamadores caem para calcular em vez de bloquear
para sempre. Veja o [guia de Remember e locks](../guias/remember-e-locks.md).
