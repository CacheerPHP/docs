# Install CacheerPHP for Laravel

This page installs CacheerPHP for Laravel in a Laravel 12 or 13 application, checks how your cache stores run, and adds a store on CacheerPHP's own storage if you want one.

## 1. Install the package

From your application root:

```sh
composer require cacheerphp/laravel
```

Laravel discovers the service provider automatically. The package installs CacheerPHP 6.1.1 or later as a dependency.

## 2. Check your stores

Your existing stores now run through CacheerPHP, with no configuration. To see which ones:

```sh
php artisan cacheer:doctor
```

```text
+----------+----------+------------------------------------------------------------------------+
| Store    | Driver   | CacheerPHP                                                             |
+----------+----------+------------------------------------------------------------------------+
| array    | array    | not routed: it lacks add() or locks, which routing must mirror exactly |
| database | database | routed                                                                 |
| file     | file     | routed                                                                 |
| redis    | redis    | routed                                                                 |
| failover | failover | not routed: it wraps other stores, which are routed themselves         |
+----------+----------+------------------------------------------------------------------------+
```

A routed store returns what it returned before and keeps its data. A store that is not routed is left exactly as it was. See [configuration](./configuration.md#routing-your-existing-stores) for why some stores stay native, and how to turn routing off.

To watch the traffic, install [Cacheer Monitor](../cacheer-monitor/quick-start.md): every routed store and every `cacheer` store reports to it with no further setup.

If that is all you need, you are done. The next steps add a store on CacheerPHP's own storage, for failover, tiered caching, or encryption.

## 3. Add a `cacheer` store

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

## 4. Make it the default

In `.env`:

```sh
CACHE_STORE=cacheer
```

Run `php artisan config:clear` after changing it. On deployments that cache
configuration, rebuild it with `php artisan config:cache`.

An existing `CACHE_STORE=database` takes precedence over the fallback value in
`env('CACHE_STORE', 'cacheer')`; change the environment value explicitly.

Everything that uses the default cache store now runs on CacheerPHP: `Cache::`, `cache()`, locks, unique jobs, the scheduler, and rate limiters. To use it for some calls only, leave the default as it is and call `Cache::store('cacheer')`.

## 5. Create the tables (database backend only)

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

## 6. Check that it works

```sh
php artisan tinker
```

```php
Cache::put('hello', 'world', 60);
Cache::get('hello');          // "world"
Cache::getStore()::class;     // "Cacheer\Laravel\CacheerStore"
```

## Switch back

To stop routing, remove the package or [turn routing off](./configuration.md#routing-your-existing-stores). Routed stores keep Laravel's own format, so nothing needs migrating.

To stop using a `cacheer` store, set `CACHE_STORE` to your previous store. CacheerPHP and Laravel use different storage layouts, so the other store starts without the entries CacheerPHP wrote; values are recomputed as they are requested.
