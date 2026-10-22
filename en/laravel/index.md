# CacheerPHP for Laravel

CacheerPHP for Laravel (`cacheerphp/laravel`) is a Laravel cache driver backed by CacheerPHP 6. Your application keeps using `Cache::`, the `cache()` helper, rate limiters, unique jobs, and the scheduler. Underneath, the store is CacheerPHP, with its failover, tiered caching, encryption at rest, and [Cacheer Monitor](../cacheer-monitor/index.md).

Requirements: PHP 8.3+, Laravel 12 or 13, and CacheerPHP 6.

## Choose how to use CacheerPHP in Laravel

| You want… | Use |
| --- | --- |
| Laravel's cache features (`Cache::`, locks, queues, scheduler) to run on CacheerPHP | **The `cacheer` driver** — this section |
| A separate CacheerPHP cache for your own services, leaving Laravel's cache as it is | [A container binding](../guides/laravel.md) — no extra package |

You can combine both: the driver also binds CacheerPHP's `Cache` contract, so services can inject it for features `Cache::` cannot express, such as tag invalidation.

## What runs on the driver

- `Cache::get()`, `put()`, `add()`, `remember()`, `flexible()`, `increment()`, `touch()`, `flush()`, and the `cache()` helper.
- Atomic locks: `Cache::lock()`, `restoreLock()`, `forceRelease()`, and `refresh()`.
- Unique jobs (`ShouldBeUnique`), the scheduler's `withoutOverlapping()`, and isolatable commands (`--isolated`).
- Rate limiting (`RateLimiter`).
- Laravel's test clock: `$this->travel()` expires entries.

`Cache::tags()` is the exception; see [differences and limitations](./limitations.md).

## Pages

- [Installation](./installation.md): install the package, add a store, and check that it works.
- [Configuration](./configuration.md): backends, failover, tiered caching, encryption, and prefixes.
- [Laravel features](./framework-features.md): locks, queues, the scheduler, rate limits, and testing.
- [Differences and limitations](./limitations.md): what behaves differently from Laravel's built-in stores, and upgrading from the 0.1 package.
