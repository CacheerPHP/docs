# CacheerPHP for Laravel

CacheerPHP for Laravel (`cacheerphp/laravel`) puts CacheerPHP underneath Laravel's cache. Install it, and the stores you already have run through CacheerPHP with the same results and the same data: your code and `config/cache.php` stay as they are, and [Cacheer Monitor](../cacheer-monitor/index.md) can show their traffic. For CacheerPHP's own storage, with failover, tiered caching, and encryption at rest, add a store that uses the `cacheer` driver.

Requirements: PHP 8.3+, Laravel 12 or 13, and CacheerPHP 6.1.1 or later, which the package installs.

## Choose how to use CacheerPHP in Laravel

| You want… | Use |
| --- | --- |
| Your existing cache stores, observed in Cacheer Monitor, with nothing else changed | **Routing**, on as soon as the package is installed |
| Failover, tiered caching, or encryption for Laravel's cache | **The `cacheer` driver**, a store on CacheerPHP's own storage |
| A separate CacheerPHP cache for your own services | [A container binding](../guides/laravel.md) |

They combine: routing runs alongside `cacheer` stores, and the driver also binds CacheerPHP's `Cache` contract, so services can inject it for features `Cache::` cannot express, such as tag invalidation.

## Routed stores

Each store Laravel builds from `config/cache.php`, such as `redis`, `database`, or `file`, runs through CacheerPHP. Reads and writes go through CacheerPHP in the store's own format and under the same keys, so existing entries stay readable and removing the package needs no flush. Everything else — `add()`, counters, `touch()`, locks, and `Cache::tags()` — goes straight to the store, so Laravel behaves exactly as before.

A few stores stay native, because routing could not reproduce them exactly; see [configuration](./configuration.md#routing-your-existing-stores).

## What runs on the `cacheer` driver

- `Cache::get()`, `put()`, `add()`, `remember()`, `flexible()`, `increment()`, `touch()`, `flush()`, and the `cache()` helper.
- Atomic locks: `Cache::lock()`, `restoreLock()`, `forceRelease()`, and `refresh()`.
- Unique jobs (`ShouldBeUnique`), the scheduler's `withoutOverlapping()`, and isolatable commands (`--isolated`).
- Rate limiting (`RateLimiter`).
- Laravel's test clock: `$this->travel()` expires entries.

`Cache::tags()` is the exception; see [differences and limitations](./limitations.md).

## Pages

- [Installation](./installation.md): install the package, check how your stores run, and add a `cacheer` store.
- [Configuration](./configuration.md): routing, backends, failover, tiered caching, encryption, and prefixes.
- [Laravel features](./framework-features.md): locks, queues, the scheduler, rate limits, and testing.
- [Differences and limitations](./limitations.md): what behaves differently from Laravel's built-in stores, on routed stores and on the `cacheer` driver, and upgrading from the 0.1 package.
- [Contributing](../contributing/laravel.md): development setup, package checks, and Redis integration tests.
