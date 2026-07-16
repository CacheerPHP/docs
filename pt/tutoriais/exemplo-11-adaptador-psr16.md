# Exemplo 11 — Adaptador PSR-16 SimpleCache

*Novo na v5.0.0*

O CacheerPHP inclui um adaptador PSR-16 que permite usá-lo em qualquer lugar onde se espera a interface padrão `\Psr\SimpleCache\CacheInterface`.

## Uso Básico

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Psr\Psr16CacheAdapter;

$cacheer = new Cacheer(['cacheDir' => __DIR__ . '/cache']);
$cache   = new Psr16CacheAdapter($cacheer);

// Guardar e obter
$cache->set('greeting', 'Olá, PSR-16!', 3600);
echo $cache->get('greeting');          // Olá, PSR-16!
echo $cache->has('greeting');          // true

// Valores por omissão para misses
echo $cache->get('missing', 'fallback');  // fallback
```

## Operações em Lote

```php
$cache->setMultiple([
    'user:1' => ['name' => 'Alice'],
    'user:2' => ['name' => 'Bob'],
], 1800);

$users = $cache->getMultiple(['user:1', 'user:2', 'user:99'], 'NOT FOUND');
// user:99 => 'NOT FOUND'

$cache->deleteMultiple(['user:1', 'user:2']);
```

## Isolamento por Namespace

Instâncias diferentes do adaptador podem partilhar o mesmo backend Cacheer mantendo as chaves separadas:

```php
$users    = new Psr16CacheAdapter($cacheer, 'users');
$sessions = new Psr16CacheAdapter($cacheer, 'sessions');

$users->set('config', 'user-value');
$sessions->set('config', 'session-value');

echo $users->get('config');     // user-value
echo $sessions->get('config');  // session-value
```

## Validação de Chaves

A PSR-16 proíbe certos caracteres nas chaves. Chaves inválidas lançam `CacheInvalidArgumentException`:

```php
use Silviooosilva\CacheerPhp\Exceptions\CacheInvalidArgumentException;

try {
    $cache->get('bad:key');
} catch (CacheInvalidArgumentException $e) {
    echo $e->getMessage();
}
```

Ver a [referência completa do Adaptador PSR-16](../api/psr16-adapter.md).
