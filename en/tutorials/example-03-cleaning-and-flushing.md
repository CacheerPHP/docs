# Cleaning and flushing

Three levels of removal: one key, one scope, or the whole store.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

// One key.
$cache->delete('user:123');

// Many keys at once.
$cache->deleteMany(['user:1', 'user:2', 'user:3']);

// One scope only.
$cache->scope('reports')->clear();

// The entire store keyspace.
$cache->clear();
```

- `delete()` returns `true` when something was removed.
- `clear()` on a **scoped** cache removes only that scope; on the root cache it
  empties the whole store. It only ever touches CacheerPHP's own keyspace.
- Expired entries are removed lazily on read, and in bulk with
  [`prune()`](./example-09-auto-flush.md).

See [Scopes](./example-04-namespaces.md).
