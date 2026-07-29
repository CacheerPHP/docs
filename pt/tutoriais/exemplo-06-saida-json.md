# O objeto CacheEntry

> A v5 tinha formatadores de saída (`toJson`, `toArray`, `toString`). A v6 guarda
> valores sem perdas — você recebe de volta exatamente o que colocou. Quando precisa de
> metadados de uma entrada, use `entry()`.

`get()` retorna o valor; `entry()` retorna um `CacheEntry` com hit/miss, timestamps e
TTL restante.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Support\SystemClock;

$cache = Cache::inMemory();
$cache->set('user:42', ['id' => 42, 'name' => 'Ada'], ttl: '10 minutes');

$entry = $cache->entry('user:42');

$entry->isHit();                          // true
$entry->value();                          // ['id' => 42, 'name' => 'Ada']
$entry->createdAt();                      // timestamp unix da escrita
$entry->expiresAt();                      // timestamp unix, ou null se para sempre
$entry->remainingTtl(new SystemClock());  // segundos restantes, ou null se para sempre
```

A formatação é responsabilidade da sua aplicação agora:

```php
$json = json_encode($cache->get('user:42'));
```

`entry()` também é como você distingue um **`null` armazenado** de um **miss** — veja
[Valores falsy e null](./exemplo-13-valores-falsy.md).
