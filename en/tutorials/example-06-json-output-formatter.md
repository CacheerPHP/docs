# Formatting cached values (JSON, array, object, string)

v6 stores values **losslessly** — `get()` gives back exactly what you put in. When
you want to reshape a value on the way out, use the `CacheDataFormatter`. There are
two ways to reach it; pick whichever reads best.

## 1. Wrap any value

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Support\CacheDataFormatter;

$cache = Cacheer::inMemory();
$cache->set('user:1', ['id' => 1, 'name' => 'Ada']);

$fmt = new CacheDataFormatter($cache->get('user:1'));

$fmt->toJson();    // '{ "id": 1, "name": "Ada" }' (pretty, UTF-8, JSON_THROW_ON_ERROR)
$fmt->toArray();   // ['id' => 1, 'name' => 'Ada']
$fmt->toObject();  // stdClass { id: 1, name: 'Ada' }
$fmt->toString();  // string cast (best for scalars)
$fmt->value();     // the raw, unformatted value
```

## 2. The fluent way — a formatted view

`->formatted()` returns a read-formatting view where `get()` returns a
`CacheDataFormatter` directly, so you can chain in one line:

```php
$cache = Cacheer::file(__DIR__ . '/cache')->formatted();

$json = $cache->get('user:1')->toJson();
$arr  = $cache->get('user:1')->toArray();
```

The view also formats `remember()`, proxies `set`/`has`/`delete`/`scope`, and
`raw()` hands back the underlying cache:

```php
$json = $cache->remember('report', 60, fn () => build_report())->toJson();

$cache->set('k', 'v');          // writes pass straight through
$raw = $cache->raw()->get('k'); // 'v' — the unformatted value
```

## Why `get()` itself isn't the formatter

The base `Cacheer::get()` must return raw values — a cached `false`, `null`, or
`0` has to come back as-is, and the PSR-16/PSR-6 adapters depend on it. So
formatting is **opt-in** via the standalone wrapper or the `formatted()` view,
never a hidden mode on `get()`.

> `toJson()` throws a `\JsonException` on an unencodable value instead of silently
> returning `false`. `toString()` follows PHP's cast rules — ideal for scalars.
