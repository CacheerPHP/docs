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

Run `php artisan config:clear` after changing it. On deployments that cache
configuration, rebuild it with `php artisan config:cache`.

Everything that uses the default cache store now runs on CacheerPHP: `Cache::`, `cache()`, locks, unique jobs, the scheduler, and rate limiters. To use it for some calls only, leave the default as it is and call `Cache::store('cacheer')`.

## 4. Create the tables (database backend only)

Skip this step unless a `cacheer` store uses `'backend' => 'database'`, as its primary store or its fallback. Publish the migration:

```sh
php artisan vendor:publish --tag=cacheer-migrations
```

Before running it, edit the published migration's `$tables` list to match your
database backends, including fallbacks. Its default is `[[null, 'cacheer_store']]`
on Laravel's default database connection. For example:

```php
private array $tables = [
    ['mysql', 'cacheer_store'],
    ['mysql', 'cacheer_fallback'],
];
```

List each connection/table pair once, then run `php artisan migrate`. Keep this
list fixed afterward so rollback reverses the original migration even when
`config/cache.php` changes. Use a new migration when adding another database
backend.

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
Failover events require CacheerPHP 6.1 and a compatible Monitor version. Ordinary
cache events also work with CacheerPHP 6.0.

## Switch back

Set `CACHE_STORE` to your previous store. CacheerPHP and Laravel use different storage layouts, so the other store starts without the entries CacheerPHP wrote; values are recomputed as they are requested.
