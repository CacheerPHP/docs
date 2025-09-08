## (Tagging e Invalidação por Tag)

Este exemplo mostra como associar chaves a tags e invalidá-las em lote em todos os drivers. Você pode usar chaves simples (sem namespace) ou chaves com namespace no formato `namespace:key`.

### File driver

```php
<?php
require __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cacheDir = __DIR__ . '/../.cache';

$cache = new Cacheer([
    'drive'    => 'file',
    'cacheDir' => $cacheDir,
]);

// Armazenar valores
$cache->putCache('user:1', ['id' => 1, 'name' => 'Ana']);
$cache->putCache('user:2', ['id' => 2, 'name' => 'Bruno']);

// Tag sem namespace explícito
$cache->tag('users', 'user:1', 'user:2');

// Chaves com namespace
$cache->putCache('profile', ['bio' => 'Hi'], 'nsA');
$cache->putCache('settings', ['lang' => 'pt'], 'nsA');
$cache->tag('ns-group', 'nsA:profile', 'nsA:settings');

// Invalidação por tag
$cache->flushTag('users');     // remove user:1 e user:2
$cache->flushTag('ns-group');  // remove nsA:profile e nsA:settings

```

Notas:
- O índice de tags é salvo em `cacheDir/_tags/{tag}.json` e removido em `flushTag`.

### Redis driver

```php
<?php
require __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useRedisDriver();

$cache->putCache('order:10', ['id' => 10]);
$cache->putCache('order:11', ['id' => 11]);
$cache->tag('orders', 'order:10', 'order:11');

// namespaced
$cache->putCache('summary', ['c' => 2], 'nsB');
$cache->tag('reports', 'nsB:summary');

$cache->flushTag('orders');  // clears order:10, order:11
$cache->flushTag('reports'); // clears nsB:summary
```

Notas:
- Usa um conjunto Redis `tag:{tag}` para indexar membros; o conjunto é excluído em `flushTag`.

### Database driver (MySQL/SQLite/PgSQL)

```php
<?php
require __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Core\Connect;

$cache = new Cacheer();
$cache->setConfig()->setDatabaseConnection(Connect::getConnection());
$cache->setDriver()->useDatabaseDriver();

$cache->putCache('p:1', ['id' => 1]);
$cache->putCache('p:2', ['id' => 2]);
$cache->tag('products', 'p:1', 'p:2');

// namespaced
$cache->putCache('view', ['ct' => 5], 'nsC');
$cache->tag('analytics', 'nsC:view');

$cache->flushTag('products');
$cache->flushTag('analytics');
```

Notas:
- O índice é armazenado na mesma tabela usando o namespace reservado `__tags__` e a chave `tag:{tag}`. Ele é removido em `flushTag`.

### Array driver (em - memória)

```php
<?php
require __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useArrayDriver();

$cache->putCache('x', 1);
$cache->putCache('y', 2);
$cache->tag('simple', 'x', 'y');
$cache->flushTag('simple');
```

Notas:
- O índice de tags vive na memória e é redefinido em `flushCache()`.
