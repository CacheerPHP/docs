# Tagging

Tags are cross-cutting labels: an entry can carry many tags, and you invalidate a
whole group at once. Tagging is a store capability (`TaggableStore`) that you call
**on the cache**, so this cache's scope is applied for you.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

$cache->set('product:1', $p1);
$cache->set('product:2', $p2);

// Associate already-stored keys with a tag.
$cache->tag('product:1', 'products', 'catalog');
$cache->tag('product:2', 'products');

// Invalidate everything tagged "products".
$removed = $cache->flushTag('products'); // number of entries removed
```

## Tags are namespaced by scope

Two scopes can use the same tag name without flushing each other:

```php
$cache->in('tenant-a')->set('p1', $a);
$cache->in('tenant-a')->tag('p1', 'products');

$cache->in('tenant-b')->set('p1', $b);
$cache->in('tenant-b')->tag('p1', 'products');

$cache->in('tenant-a')->flushTag('products');
$cache->in('tenant-b')->get('p1');   // still $b
```

Notes:

- Tagging associates **existing** keys with a tag; store the value first.
- Tag indexes are best-effort metadata: a key that expires before its tag is
  flushed is simply a no-op.
- On a store that is not taggable, `tag()`/`flushTag()` throw
  `UnsupportedCapabilityException`. Ask first with
  `$cache->supports(TaggableStore::class)` if your backend is pluggable.
- For structural separation (features, tenants), prefer
  [scopes](./example-04-namespaces.md); use tags for relationships that cut across
  scopes.
