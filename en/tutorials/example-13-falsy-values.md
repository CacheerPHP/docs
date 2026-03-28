# Example 13 — Caching Falsy Values

*New in v5.0.0*

CacheerPHP v5.0.0 correctly stores and retrieves **falsy** values such as `0`, `''`, `false`, and `[]`. In previous versions these were mistakenly treated as cache misses.

## The Problem (v4.x)

```php
$cache->putCache('counter', 0);
$value = $cache->getCache('counter');
// v4: $value was treated as "not found" because of !empty() checks
```

## The Fix (v5.0.0)

v5 uses `isSuccess()` internally instead of `!empty()`, so any PHP value — including falsy ones — is returned correctly.

## Storing and Retrieving Falsy Values

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useArrayDriver();

// Integer zero
$cache->putCache('zero', 0);
echo $cache->getCache('zero'); // 0

// Empty string
$cache->putCache('empty_str', '');
var_dump($cache->getCache('empty_str')); // string(0) ""

// Boolean false
$cache->putCache('flag', false);
var_dump($cache->getCache('flag')); // bool(false)

// Empty array
$cache->putCache('list', []);
var_dump($cache->getCache('list')); // array(0) {}
```

## remember() with Falsy Values

The `remember()` method no longer re-executes the callback when the cached value is falsy:

```php
$callCount = 0;

$value = $cache->remember('counter', 3600, function () use (&$callCount) {
    $callCount++;
    return 0; // falsy but valid
});

// Second call — callback is NOT invoked again
$value = $cache->remember('counter', 3600, function () use (&$callCount) {
    $callCount++;
    return 0;
});

echo $callCount; // 1 (not 2)
echo $value;     // 0
```

## increment() and decrement() from Zero

```php
$cache->putCache('hits', 0);

$cache->increment('hits');
echo $cache->getCache('hits'); // 1

$cache->putCache('balance', 0);
$cache->decrement('balance');
echo $cache->getCache('balance'); // -1
```
