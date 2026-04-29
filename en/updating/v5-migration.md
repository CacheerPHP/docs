# Upgrading to v5.0.0

This guide covers every breaking change, new feature, and the steps you need to take when upgrading from CacheerPHP v4.x to v5.0.0.

---

## Requirements Change

| | v4.x | v5.0.0 |
|-|------|--------|
| PHP | >= 8.0 | **>= 8.2** |
| psr/simple-cache | — | ^3.0 (new) |
| psr/log | — | ^3.0 (new) |

Update your `composer.json`:

```sh
composer require silviooosilva/cacheer-php:^5.0
```

---

## Breaking Changes

### 1. `$cacheStore` and `$options` are now private

Direct property access no longer works:

```php
// v4 (removed)
$cache->cacheStore;
$cache->options;
$cache->options['loggerPath'] = 'cacheer.log';

// v5 (use accessors)
$cache->getCacheStore();
$cache->getOptions();
$cache->setOption('loggerPath', 'cacheer.log');
$cache->setOptions(['cacheDir' => '/tmp/cache']);
```

### 2. `add()` return value is corrected

The return values were **inverted** in v4. v5 matches the standard convention:

```php
// v4 (wrong)
$cache->add('existing_key', 'data');  // returned true
$cache->add('new_key', 'data');       // returned false

// v5 (correct)
$cache->add('existing_key', 'data');  // returns false (key exists, nothing written)
$cache->add('new_key', 'data');       // returns true  (key is new, value stored)
```

**Action required:** If your code checks `add()` return values, invert the logic.

### 3. `CACHE_FOREVER_TTL` changed from `31536000000` to `PHP_INT_MAX`

The old value (`31536000 * 1000`) could overflow on 32-bit systems. The new value is `PHP_INT_MAX`.

**Action required:** None for most users. If you compare TTL values against the old constant, update them.

### 4. Encryption format changed (random IV)

v4 used a deterministic IV derived from the key hash, making identical payloads produce identical ciphertexts. v5 generates a fresh random IV per write and prepends it to the ciphertext.

**Action required:** Data encrypted with v4 **cannot** be decrypted by v5. Flush encrypted caches before upgrading, or write new values to re-encrypt them.

### 5. FileCacheStore envelope format

v5 stores per-item TTL in a JSON envelope `{data, expires_at, ttl}` instead of relying on file modification time. v5 includes a legacy fallback that reads v4-format files, so existing caches will continue to work until they expire naturally.

**Action required:** None — the fallback handles it. To force the new format immediately, flush the file cache after upgrading.

### 6. `CacheDataFormatter::toJson()` throws on failure

The return type changed from `string|false` to `string`, and the method now uses `JSON_THROW_ON_ERROR`. If encoding fails, a `\JsonException` is thrown instead of returning `false`.

**Action required:** If your code checked for `false` from `toJson()`, wrap the call in a try/catch instead.

### 7. `Cacheer::getOptions()` cannot be called statically

Because `getOptions()` is now a public instance method on `Cacheer`, PHP's method resolution prevents calling it via `__callStatic`. Use the `setConfig()` route:

```php
// v4
Cacheer::getOptions();

// v5
Cacheer::setConfig()->getOptions();
// or on an instance:
$cache->getOptions();
```

---

## New Features

### PSR-16 SimpleCache Adapter

```php
use Silviooosilva\CacheerPhp\Psr\Psr16CacheAdapter;

$psr = new Psr16CacheAdapter($cache);
$psr->set('key', 'value', 3600);
$psr->get('key', 'default');
$psr->getMultiple(['a', 'b', 'c']);
```

See [PSR-16 Adapter reference](../api/psr16-adapter.md).

### PSR-3 Logger

`CacheLogger` now extends `\Psr\Log\AbstractLogger`, making it compatible with any PSR-3 consumer.

### DateInterval TTL

All methods that accept a TTL now also accept `\DateInterval` and `null`:

```php
$cache->putCache('key', 'data', ttl: new \DateInterval('PT30M'));
$cache->putCache('key', 'data', ttl: null);  // forever
```

### Falsy Value Caching

`0`, `0.0`, `''`, `'0'`, `false`, and `[]` are now valid cache hits. The library uses `isSuccess()` instead of `!empty()` internally, so `remember()`, `increment()`, and `getAndForget()` work correctly with these values.

### Instance Management

```php
$cache->stats();              // ['driver' => '...', 'compression' => bool, 'encryption' => bool]
Cacheer::resetInstance();     // clears the static singleton
Cacheer::setInstance($cache); // replaces the static singleton
```

### New Accessors

```php
$cache->getCacheStore();             // returns the active CacheerInterface driver
$cache->setCacheStore($newDriver);   // switches the driver at runtime
$cache->getOptions();                // returns the options array
$cache->setOption('key', 'value');   // sets a single option
$cache->setOptions([...]);           // replaces all options
```

---

## Migration Checklist

1. Update PHP to 8.2+
2. Run `composer require silviooosilva/cacheer-php:^5.0`
3. Search your codebase for `->cacheStore` and `->options` — replace with accessor methods
4. Search for `->add()` calls — verify the return value logic is correct for the new semantics
5. If you use encryption, flush encrypted caches before deploying
6. If you call `Cacheer::getOptions()` statically, switch to `Cacheer::setConfig()->getOptions()`
7. If you check `toJson() === false`, switch to a try/catch for `\JsonException`
8. Run your test suite: `vendor/bin/phpunit`
