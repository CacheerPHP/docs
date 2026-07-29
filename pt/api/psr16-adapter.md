# Adaptadores PSR-16 e PSR-6

O CacheerPHP 6 traz adaptadores para os dois PSRs de cache sobre o mesmo núcleo
`Cache`, para você entregar um cache padronizado a qualquer biblioteca
interoperável.

## PSR-16 — `Psr16Cache`

`Silviooosilva\CacheerPhp\Psr\Psr16Cache` implementa
`Psr\SimpleCache\CacheInterface`.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Psr\Psr16Cache;

$psr16 = new Psr16Cache(Cache::file('/var/cache/app'));

$psr16->set('token', 'abc123', 1800);        // ttl: int segundos, DateInterval ou null
$psr16->get('token');                         // 'abc123'
$psr16->get('missing', 'default');            // 'default'
$psr16->has('token');                         // true
$psr16->delete('token');

$psr16->setMultiple(['a' => 1, 'b' => 2], 3600);
$psr16->getMultiple(['a', 'b', 'c'], 'default');
$psr16->deleteMultiple(['a', 'b']);
$psr16->clear();
```

Detalhes de spec honrados:

- Caracteres reservados na chave (`{}()/\@:`) lançam `CacheInvalidArgumentException`
  (que implementa a `InvalidArgumentException` do PSR-16 *e* do PSR-6).
- TTL `null` significa para sempre; `<= 0` apaga a chave (como a spec exige).
- Um `null` cacheado é retornado como hit, distinto do padrão num miss.

## PSR-6 — `Psr6Pool`

`Silviooosilva\CacheerPhp\Psr\Psr6Pool` implementa
`Psr\Cache\CacheItemPoolInterface`; os itens são `Psr6Item`. O pool recebe um
`Cache` e um `Clock`.

```php
use Silviooosilva\CacheerPhp\Psr\Psr6Pool;
use Silviooosilva\CacheerPhp\Support\SystemClock;

$pool = new Psr6Pool(Cache::file('/var/cache/app'), new SystemClock());

$item = $pool->getItem('user:42');
if (! $item->isHit()) {
    $item->set($user)->expiresAfter(600); // segundos ou um DateInterval
    $pool->save($item);
}
$user = $pool->getItem('user:42')->get();

// Saves deferidos são descarregados no commit().
$pool->saveDeferred($pool->getItem('a')->set(1));
$pool->commit();
```

`Psr6Item::expiresAfter()` é agnóstico ao clock (relativo), enquanto
`expiresAt(DateTimeInterface)` fixa uma expiração absoluta; o pool resolve ambos
contra o clock injetado, então a expiração PSR-6 é determinística sob um `FakeClock`.

## Qual usar?

- Use **PSR-16** para cache chave/valor direto e o maior suporte de bibliotecas.
- Use **PSR-6** quando uma biblioteca exigir o modelo pool/item ou commits deferidos.
- Use a API nativa [`Cache`](./funcoes-cache.md) para escopos, `remember()`,
  `flexible()`, políticas, camadas e resiliência — recursos que os PSRs não cobrem.
