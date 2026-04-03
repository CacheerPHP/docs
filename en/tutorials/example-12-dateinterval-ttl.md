# Example 12 — DateInterval and null TTL

*New in v5.0.0*

All TTL parameters now accept four formats: `int` (seconds), `string` (human-readable), `\DateInterval`, and `null` (forever).

## DateInterval TTL

Use PHP's native `\DateInterval` for type-safe, IDE-friendly TTLs:

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useArrayDriver();

// 45 minutes
$cache->putCache('session', $data, ttl: new DateInterval('PT45M'));

// 1 day
$cache->putCache('report', $data, ttl: new DateInterval('P1D'));

// 2 hours 30 minutes
$cache->putCache('token', $data, ttl: new DateInterval('PT2H30M'));
```

## null TTL — Store Forever

Passing `null` stores the item with no expiration (`PHP_INT_MAX` seconds):

```php
$cache->putCache('app_version', 'v5.0.0', ttl: null);
```

## Works Everywhere

`\DateInterval` and `null` are accepted by all methods that take a TTL:

```php
// putCache
$cache->putCache('key', 'data', ttl: new DateInterval('PT30M'));

// add (conditional put)
$cache->add('lock', getmypid(), ttl: new DateInterval('PT1M'));

// remember (get-or-compute)
$cache->remember('stats', new DateInterval('PT5M'), fn() => computeStats());

// renewCache (extend TTL)
$cache->renewCache('key', new DateInterval('PT2H'));
```

## Comparison of All Formats

```php
// Integer — seconds
$cache->putCache('k', 'v', ttl: 3600);

// String — human-readable
$cache->putCache('k', 'v', ttl: '2 hours');

// DateInterval — PHP native
$cache->putCache('k', 'v', ttl: new DateInterval('PT2H'));

// null — forever
$cache->putCache('k', 'v', ttl: null);
```
