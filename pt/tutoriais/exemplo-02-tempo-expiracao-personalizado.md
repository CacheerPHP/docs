# Expiração personalizada (TTL)

Todo método que guarda um valor aceita as mesmas formas de TTL.

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Kernel\Ttl;

$cache = Cacheer::inMemory();

$cache->set('a', $v, ttl: 3600);                     // int segundos
$cache->set('b', $v, ttl: '2 hours');                // string legível
$cache->set('c', $v, ttl: new DateInterval('PT30M')); // DateInterval
$cache->set('d', $v, ttl: Ttl::minutes(15));         // objeto Ttl
$cache->set('e', $v, ttl: null);                     // para sempre
$cache->set('f', $v, ttl: 'forever');                // para sempre (string)
```

Strings legíveis aceitam `<n> second|minute|hour|day|week` (singular ou plural).

## Lendo o TTL restante

Use `entry()` para ver quanto falta:

```php
use Silviooosilva\CacheerPhp\Support\SystemClock;

$entry = $cache->entry('b');
echo $entry->remainingTtl(new SystemClock()) ?? 'nunca'; // segundos, ou null se para sempre
```

## Notas

- `Ttl::seconds()` exige um valor **maior que zero** — não existe regra de "zero
  apaga"; apague explicitamente com `delete()`.
- "Para sempre" é guardado como "sem expiração", não um inteiro gigante.

Veja [TTL e Clock](../api/construtor-de-tempo.md) para a referência completa.
