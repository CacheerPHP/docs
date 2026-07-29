# Tagging

Tags are cross-cutting labels: an entry can carry many tags, and you invalidate a
whole group at once. Tagging is a store capability (`TaggableStore`), so you reach
it through the store.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Kernel\Key;
use Silviooosilva\CacheerPhp\Stores\FileStore;

$store = new FileStore(__DIR__ . '/cache');
$cache = new Cache($store);

$cache->set('product:1', $p1);
$cache->set('product:2', $p2);

// Associate already-stored keys with a tag.
$store->tag(Key::named('product:1'), 'products', 'catalog');
$store->tag(Key::named('product:2'), 'products');

// Invalidate everything tagged "products".
$removed = $store->clearTag('products'); // number of entries removed
```

Notes:

- Tagging associates **existing** keys with a tag; store the value first.
- Tag indexes are best-effort metadata: a key that expires before its tag is
  flushed is simply a no-op.
- For structural separation (features, tenants), prefer
  [scopes](./example-04-namespaces.md); use tags for relationships that cut across
  scopes.
