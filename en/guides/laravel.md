# Bind CacheerPHP in Laravel services

Bind CacheerPHP in the service container and inject its `Cache` contract into application services. This guide targets Laravel 13.x and CacheerPHP 6.x. The binding adds a cache for your services; Laravel's existing cache configuration continues to control its own framework features.

The package this guide installs, [CacheerPHP for Laravel](../laravel/index.md), also routes Laravel's own stores through CacheerPHP with unchanged results, and adds a `cacheer` driver that runs Laravel's cache features (`Cache::`, locks, queues, the scheduler) on CacheerPHP's own storage. This guide adds a separate cache for your services.

First, [compare the two cache APIs](./cacheerphp-vs-laravel-cache.md) if you are deciding whether to add CacheerPHP. For a runnable standalone example, use [getting started](../getting-started/index.md).

## 1. Install

From your Laravel application root:

```sh
composer require cacheerphp/laravel
```

## 2. Bind the cache

Add this binding to the existing `register()` method in `app/Providers/AppServiceProvider.php`, preserving your other bindings:

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Contracts\Cache;

public function register(): void
{
    $this->app->scoped(Cache::class, static fn (): Cache => Cacheer::build()
        ->file(storage_path('framework/cache/cacheer-v6'))
        ->create());
}
```

The file store persists across requests on one host. The `scoped()` container lifetime gives each request or job its own cache instance in supported Laravel worker lifecycles. See [Laravel's scoped bindings](https://laravel.com/docs/13.x/container#binding-scoped-singletons). Keep this directory writable and outside `public/`. For multiple hosts, choose a shared Redis or database store through the same binding.

## 3. Inject it into a service

For a public product catalog, create `app/Services/ProductCatalog.php`. This example assumes an existing `App\Models\Product` model with `id`, `name`, and `price` columns:

```php
<?php

namespace App\Services;

use App\Models\Product;
use Silviooosilva\CacheerPhp\Contracts\Cache;

final class ProductCatalog
{
    public function __construct(private readonly Cache $cache)
    {
    }

    public function all(): array
    {
        return $this->cache->scope('catalog')->remember(
            'products',
            300,
            static fn (): array => Product::query()
                ->get(['id', 'name', 'price'])
                ->toArray(),
        );
    }

    public function invalidate(): void
    {
        $this->cache->scope('catalog')->delete('products');
    }
}
```

Inject `ProductCatalog` into your controller, or try a route in `routes/web.php`:

```php
use App\Services\ProductCatalog;
use Illuminate\Support\Facades\Route;

Route::get('/catalog', static fn (ProductCatalog $catalog): array => $catalog->all());
```

The first request queries the database; subsequent requests reuse the result for five minutes. Call `$catalog->invalidate()` after a successful product update and after its transaction commits. The next request reloads the catalog.

This sample is for shared public data. For tenant-specific data, resolve authorization before reading the cache and include the authorized tenant and query filters in the scope/key.

## 4. Verify the integration

Request `/catalog` twice and inspect the database query log or [Cacheer Monitor](../cacheer-monitor/quick-start.md). In a test, inject `Cacheer::inMemory()` for fast local storage. The same `ProductCatalog` service continues to depend on the native cache contract.

For stale-while-revalidate, configure a [deferred executor](./stale-while-revalidate.md) for your Laravel lifecycle: the default CacheerPHP executor is synchronous. For configuration beyond files, see [stores and capabilities](../api/drivers.md), [policies](./policies.md), and [observability](./observability.md).
