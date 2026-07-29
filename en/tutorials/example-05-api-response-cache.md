# Caching an API response

`remember()` returns the cached value, or computes it once on a miss and stores it.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;

$cache = Cache::file(__DIR__ . '/cache');

$weather = $cache->remember('weather:lisbon', ttl: '15 minutes', callback: function () {
    $json = file_get_contents('https://api.example.com/weather?city=lisbon');
    return json_decode($json, true);
});
```

- On a hit, the HTTP call never happens.
- On a concurrent miss, the callback runs **once** across workers (single-flight),
  not once per request — no stampede. See
  [Stampede protection](./example-18-stampede-and-swr.md).
- Cache forever by passing `ttl: null`.

For hot endpoints where you never want a user to wait for a refresh, use
[`flexible()`](./example-18-stampede-and-swr.md) instead.
