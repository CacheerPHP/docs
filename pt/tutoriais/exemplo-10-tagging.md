# Tagging

Tags são rótulos transversais: uma entrada pode ter muitas tags, e você invalida um
grupo inteiro de uma vez. Tagging é uma capacidade da store (`TaggableStore`) que
você chama **no cache**, então o escopo deste cache é aplicado para você.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

$cache->set('product:1', $p1);
$cache->set('product:2', $p2);

// Associa chaves já armazenadas a uma tag.
$cache->tag('product:1', 'products', 'catalog');
$cache->tag('product:2', 'products');

// Invalida tudo com a tag "products".
$removed = $cache->flushTag('products'); // número de entradas removidas
```

## Tags têm namespace por escopo

Dois escopos podem usar o mesmo nome de tag sem limpar um ao outro:

```php
$cache->in('tenant-a')->set('p1', $a);
$cache->in('tenant-a')->tag('p1', 'products');

$cache->in('tenant-b')->set('p1', $b);
$cache->in('tenant-b')->tag('p1', 'products');

$cache->in('tenant-a')->flushTag('products');
$cache->in('tenant-b')->get('p1');   // continua $b
```

Notas:

- Tagging associa chaves **existentes** a uma tag; armazene o valor primeiro.
- Índices de tag são metadados de melhor esforço: uma chave que expira antes da tag
  ser limpa é simplesmente um no-op.
- Em uma store não taggeável, `tag()`/`flushTag()` lançam
  `UnsupportedCapabilityException`. Pergunte antes com
  `$cache->supports(TaggableStore::class)` se o seu backend é plugável.
- Para separação estrutural (funcionalidades, tenants), prefira
  [escopos](./exemplo-04-namespaces.md); use tags para relações que cruzam escopos.
