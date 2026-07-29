# Simple data cache

The four operations you'll use most: store, read, check, delete.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use Silviooosilva\CacheerPhp\Kernel\Cache;

$cache = Cache::file(__DIR__ . '/cache'); // or Cache::inMemory()

// Store — TTL is optional; null means forever.
$cache->set('user:123', ['id' => 123, 'name' => 'John Doe'], ttl: '10 minutes');

// Read — returns your default on a miss.
$user = $cache->get('user:123', default: null);
print_r($user);

// Check existence.
if ($cache->has('user:123')) {
    echo "cache hit\n";
}

// Delete a single key.
$cache->delete('user:123');
```

Notes:

- `get()` returns the value directly, or the `$default` you pass on a miss.
- Any serializable value works — scalars, arrays, and objects. See
  [Falsy and null values](./example-13-falsy-values.md) for the one subtlety.
- Swap `Cache::file(...)` for `Cache::inMemory()`, `Cache::redis(...)`, or
  `Cache::database(...)` without changing the rest of your code.

Next: [Custom expiration (TTL)](./example-02-custom-expiration-ttl.md).
