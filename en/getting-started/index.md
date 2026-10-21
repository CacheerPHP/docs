# Getting Started

**Framework-independent PHP caching with scoped invalidation, composable stores, and optional monitoring.** This walkthrough caches a small product catalog, shows a hit across two PHP processes, and refreshes the result when the source changes.

## 1. Install

You need **PHP 8.3+** and Composer. File storage needs no Redis server, database, or optional PHP extensions.

```sh
mkdir cacheer-demo
cd cacheer-demo
composer require silviooosilva/cacheer-php:"^6.0"
```

Already have a PHP project? Run only the Composer command from its root. The `^6.0` constraint stays within stable 6.x releases.

## 2. Add a source and a cache

Create `products.json` in the project root:

```json
[
  {"id": 1, "name": "Desk lamp"},
  {"id": 2, "name": "Notebook"}
]
```

Save this as `cache-demo.php` beside it:

```php
<?php

declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$catalog = Cacheer::file(__DIR__ . '/var/cache')->scope('catalog');

if (in_array('--refresh', $argv, true)) {
    $catalog->delete('products');
}

$products = $catalog->remember('products', 300, static function (): array {
    echo "Reading products.json...\n";

    return json_decode(
        file_get_contents(__DIR__ . '/products.json'),
        true,
        flags: JSON_THROW_ON_ERROR,
    );
});

echo 'Products: ' . count($products) . PHP_EOL;
```

The callback is your data source. Replace the JSON read with a database query or an API call in your application. Construct the cache once in bootstrap code and inject it into the services that need it. Keep `var/cache` writable and outside the public web root.

## 3. See a miss, then a hit

```sh
php cache-demo.php
php cache-demo.php
```

First run:

```text
Reading products.json...
Products: 2
```

Second run, within five minutes:

```text
Products: 2
```

The second process reads the stored result without running the callback. The file store persists between requests and CLI runs. `Cacheer::inMemory()` lasts only for the lifetime of its PHP process.

## 4. Change the source and invalidate

Add a third product to `products.json`. Run `php cache-demo.php`: it still reports two products while the cached result is valid.

```sh
php cache-demo.php --refresh
```

This deletes only `products` in the `catalog` scope, so the callback reads the updated file and reports three products. In an application, invalidate the key after the source update succeeds. `$catalog->clear()` clears that scope; keep a root cache if you also need to clear the whole configured store.

## Continue in your application

| Your application | Next step |
| --- | --- |
| Vanilla PHP or a worker | Create the cache in bootstrap and inject `Contracts\Cache` into your services. The example above is ready to run. |
| Laravel | [Bind the cache in a service provider and inject it into a catalog service](../guides/laravel.md). |
| Still deciding | [Compare CacheerPHP with Laravel Cache](../guides/cacheerphp-vs-laravel-cache.md). |
| Want to inspect hits and misses | [Connect Cacheer Monitor](../cacheer-monitor/quick-start.md). |

## Choose storage when you need it

Start with files on one host. Choose [Redis](../api/drivers.md) or a [database store](../api/drivers.md) when processes on multiple hosts need a shared backend. Supply the connection yourself; create a database store's schema explicitly during setup. For tests and short CLI tasks, use `Cacheer::inMemory()`.

## A few rules to keep

- `get()` returns the original value. Cached `null`, `false`, `0`, and empty arrays are valid hits; use `has()` or `entry()->isHit()` when presence matters.
- TTL accepts positive seconds, a duration string such as `'5 minutes'`, a `DateInterval`, or `Ttl`. An omitted/null TTL uses the bound default policy, otherwise forever. Use `forever()` for explicit no-expiry storage.
- `remember()` coordinates misses when the store can lock. Lease expiry and lock timeouts can still allow duplicate work; callbacks must tolerate that.
- `flexible()` supports stale-while-revalidate. Its default executor runs inline; [configure deferred execution](../guides/stale-while-revalidate.md) before expecting refreshes after the response.

See the [API reference](../api/index.md) for signatures, [version support](../updating/version-support.md) for maintenance, or the [migration guide](../updating/index.md) if you are coming from v5.
