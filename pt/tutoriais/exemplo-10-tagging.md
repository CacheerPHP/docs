# Tagging

Tags são rótulos transversais: uma entrada pode ter muitas tags, e você invalida um
grupo inteiro de uma vez. Tagging é uma capacidade da store (`TaggableStore`), então
você a acessa pela store.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Kernel\Key;
use Silviooosilva\CacheerPhp\Stores\FileStore;

$store = new FileStore(__DIR__ . '/cache');
$cache = new Cache($store);

$cache->set('product:1', $p1);
$cache->set('product:2', $p2);

// Associe chaves já armazenadas a uma tag.
$store->tag(Key::named('product:1'), 'products', 'catalog');
$store->tag(Key::named('product:2'), 'products');

// Invalide tudo com a tag "products".
$removed = $store->clearTag('products'); // número de entradas removidas
```

Notas:

- Tagging associa chaves **existentes** a uma tag; guarde o valor primeiro.
- Índices de tag são metadados best-effort: uma chave que expira antes de a tag ser
  limpa é simplesmente um no-op.
- Para separação estrutural (funcionalidades, tenants), prefira
  [escopos](./exemplo-04-namespaces.md); use tags para relações que cruzam escopos.
