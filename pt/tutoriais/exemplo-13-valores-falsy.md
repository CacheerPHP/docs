# Valores falsy e null

Um `null`, `false`, `0`, `''` ou `[]` cacheado é um **hit**, retornado exatamente como
armazenado. Só `entry()` consegue distinguir um `null` armazenado de um miss.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;

$cache = Cache::inMemory();

$cache->set('flag', false);
$cache->set('count', 0);
$cache->set('nothing', null);

$cache->get('flag');   // false (um hit, não um miss)
$cache->get('count');  // 0
$cache->get('nothing'); // null
```

Como `get('nothing')` e `get('absent')` retornam ambos `null`, use `entry()` quando a
diferença importar:

```php
$cache->entry('nothing')->isHit(); // true  — um null armazenado
$cache->entry('absent')->isHit();  // false — um miss real
```

Ou passe um sentinela como padrão:

```php
$missing = new stdClass();
if ($cache->get('nothing', $missing) === $missing) {
    // realmente ausente
}
```

É por isso que `remember()` não recalcula um resultado legitimamente vazio a cada
chamada — um `null` armazenado conta como hit.
