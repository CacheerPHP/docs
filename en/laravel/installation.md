# Install CacheerPHP for Laravel

This page adds a CacheerPHP store to a Laravel 12 or 13 application and makes it the default cache. Switching back later is one line in `.env`.

## 1. Install the package

From your application root:

```sh
composer require cacheerphp/laravel
```

Laravel discovers the service provider automatically. The package installs CacheerPHP 6 as a dependency.

## 2. Add a store

In `config/cache.php`, add a store that uses the `cacheer` driver:

```php
'stores' => [

    'cacheer' => [
        'driver' => 'cacheer',
        'backend' => 'file',
    ],

    // ...your existing stores
],
```

The `file` backend needs no services and keeps entries in `storage/framework/cache/cacheer`. For Redis, a database, failover, or encryption, see [configuration](./configuration.md).

## 3. Make it the default

In `.env`:

```sh
CACHE_STORE=cacheer
```

Everything that uses the default cache store now runs on CacheerPHP: `Cache::`, `cache()`, locks, unique jobs, the scheduler, and rate limiters. To use it for some calls only, leave the default as it is and call `Cache::store('cacheer')`.

## 4. Create the tables (database backend only)

Skip this step unless a `cacheer` store uses `'backend' => 'database'`, as its primary store or its fallback. Publish the migration and run it:

```sh
php artisan vendor:publish --tag=cacheer-migrations
php artisan migrate
```

The migration creates the tables of every `cacheer` store configured with the database backend, on that store's connection.

## 5. Check that it works

```sh
php artisan tinker
```

```php
Cache::put('hello', 'world', 60);
Cache::get('hello');          // "world"
Cache::getStore()::class;     // "Cacheer\Laravel\CacheerStore"
```

To watch the traffic, install [Cacheer Monitor](../cacheer-monitor/quick-start.md); every `cacheer` store reports to it with no further setup.

## Switch back

Set `CACHE_STORE` to your previous store. CacheerPHP and Laravel use different storage layouts, so the other store starts without the entries CacheerPHP wrote; values are recomputed as they are requested.
