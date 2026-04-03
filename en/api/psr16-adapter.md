# PSR-16 SimpleCache Adapter

*New in v5.0.0*

`Psr16CacheAdapter` wraps any `Cacheer` instance and exposes the standard `\Psr\SimpleCache\CacheInterface`. This lets you use CacheerPHP anywhere a PSR-16 cache is expected — framework service containers, third-party libraries, or your own typed dependencies.

## Installation

The adapter is included in the main package. The required `psr/simple-cache` ^3.0 dependency is pulled in automatically.

```sh
composer require silviooosilva/cacheer-php
```

## Basic Usage

```php
<?php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Psr\Psr16CacheAdapter;

$cacheer = new Cacheer(['cacheDir' => __DIR__ . '/cache']);
$cache   = new Psr16CacheAdapter($cacheer);

// set / get / has / delete
$cache->set('user:1', ['name' => 'Alice'], 3600);
$cache->get('user:1');               // ['name' => 'Alice']
$cache->get('missing', 'default');   // 'default'
$cache->has('user:1');               // true
$cache->delete('user:1');
```

## Constructor

```php
new Psr16CacheAdapter(Cacheer $cache, string $namespace = '')
```

| Parameter | Description |
|-----------|-------------|
| `$cache` | The underlying Cacheer instance to delegate to |
| `$namespace` | Optional namespace applied to every key (isolates different adapter instances) |

## Methods

### Single-item Operations

| Method | Signature | Description |
|--------|-----------|-------------|
| `get` | `get(string $key, mixed $default = null): mixed` | Fetch a value; returns `$default` on miss |
| `set` | `set(string $key, mixed $value, int\|DateInterval\|null $ttl = null): bool` | Store a value |
| `delete` | `delete(string $key): bool` | Remove a key |
| `has` | `has(string $key): bool` | Check existence |
| `clear` | `clear(): bool` | Flush the entire cache store |

### Batch Operations

| Method | Signature | Description |
|--------|-----------|-------------|
| `getMultiple` | `getMultiple(iterable $keys, mixed $default = null): iterable` | Fetch many keys at once |
| `setMultiple` | `setMultiple(iterable $values, int\|DateInterval\|null $ttl = null): bool` | Store many key/value pairs |
| `deleteMultiple` | `deleteMultiple(iterable $keys): bool` | Remove many keys |

## TTL Behaviour

| Input | Effect |
|-------|--------|
| `null` | Store forever (`PHP_INT_MAX` seconds) |
| Positive `int` | Seconds until expiry |
| `0` or negative `int` | Expire immediately (key is deleted) |
| `\DateInterval` | Converted to seconds automatically |

```php
$cache->set('forever', 'data', null);                      // no expiry
$cache->set('timed', 'data', 1800);                        // 30 minutes
$cache->set('interval', 'data', new DateInterval('PT1H')); // 1 hour
$cache->set('ephemeral', 'data', 0);                       // deleted immediately
```

## Key Validation

PSR-16 defines reserved characters that must **not** appear in cache keys. The adapter throws `CacheInvalidArgumentException` (which implements `\Psr\SimpleCache\InvalidArgumentException`) for:

- Empty strings
- Keys containing any of: `{ } ( ) / \ @ :`

```php
use Silviooosilva\CacheerPhp\Exceptions\CacheInvalidArgumentException;

try {
    $cache->get('bad:key');
} catch (CacheInvalidArgumentException $e) {
    // "Cache key "bad:key" contains reserved PSR-16 characters"
}
```

## Namespace Isolation

Create multiple adapters with different namespaces to isolate modules sharing the same Cacheer instance:

```php
$users    = new Psr16CacheAdapter($cacheer, 'users');
$sessions = new Psr16CacheAdapter($cacheer, 'sessions');

$users->set('config', 'user-value');
$sessions->set('config', 'session-value');

$users->get('config');    // 'user-value'
$sessions->get('config'); // 'session-value'
```

## Framework Integration Example

Bind the adapter into a Laravel-style container:

```php
$container->singleton(\Psr\SimpleCache\CacheInterface::class, function () {
    $cacheer = new Cacheer(['cacheDir' => storage_path('cache')]);
    return new Psr16CacheAdapter($cacheer);
});
```
