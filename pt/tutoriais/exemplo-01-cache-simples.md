# Cache simples de dados

As quatro operações que você mais usará: guardar, ler, checar, apagar.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache'); // ou Cacheer::inMemory()

// Guardar — o TTL é opcional; null significa para sempre.
$cache->set('user:123', ['id' => 123, 'name' => 'John Doe'], ttl: '10 minutes');

// Ler — retorna seu padrão num miss.
$user = $cache->get('user:123', default: null);
print_r($user);

// Checar existência.
if ($cache->has('user:123')) {
    echo "cache hit\n";
}

// Apagar uma chave.
$cache->delete('user:123');
```

Notas:

- `get()` retorna o valor diretamente, ou o `$default` num miss.
- Qualquer valor serializável funciona — escalares, arrays e objetos. Veja
  [Valores falsy e null](./exemplo-13-valores-falsy.md) para a única sutileza.
- Troque `Cacheer::file(...)` por `Cacheer::inMemory()`, `Cacheer::redis(...)` ou
  `Cacheer::database(...)` sem mudar o resto do seu código.

Próximo: [Expiração personalizada (TTL)](./exemplo-02-tempo-expiracao-personalizado.md).
