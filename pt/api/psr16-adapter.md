# Adaptadores PSR-16 e PSR-6

O CacheerPHP 6 traz adaptadores para os dois PSRs de cache sobre o mesmo núcleo
`Cacheer`, para você entregar um cache padronizado a qualquer biblioteca
interoperável.

## PSR-16 — `Psr16Cache`

`Silviooosilva\CacheerPhp\Psr\Psr16Cache` implementa
`Psr\SimpleCache\CacheInterface`.

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Psr\Psr16Cache;

$psr16 = new Psr16Cache(Cacheer::file('/var/cache/app'));

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

- Toda chave inválida lança `CacheInvalidArgumentException` (que implementa a
  `InvalidArgumentException` do PSR-16 *e* do PSR-6): caracteres reservados
  (`{}()/\@:`), chave vazia, chave com mais de 1.024 bytes ou com caracteres de
  controle, e uma chave que não é string numa chamada em lote. Um TTL além do limite
  da plataforma também a lança. Os dois adaptadores se comportam igual.
- Métodos de escrita (`set`, `delete`, `clear` e as variantes `*Multiple`; no PSR-6
  `save`, `deleteItem(s)`, `clear`, `commit`) retornam `false` quando o store falha,
  como as specs exigem. Leituras continuam lançando numa falha do store, em vez de
  esconder uma indisponibilidade atrás do padrão.
- TTL `null` significa para sempre; `<= 0` apaga a chave (como a spec exige).
- Um `null` cacheado é retornado como hit, distinto do padrão num miss.

## PSR-6 — `Psr6Pool`

`Silviooosilva\CacheerPhp\Psr\Psr6Pool` implementa
`Psr\Cache\CacheItemPoolInterface`; os itens são `Psr6Item`. O pool recebe um
`Cacheer` e um `Clock`.

```php
use Silviooosilva\CacheerPhp\Psr\Psr6Pool;
use Silviooosilva\CacheerPhp\Support\SystemClock;

$pool = new Psr6Pool(Cacheer::file('/var/cache/app'), new SystemClock());

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

Um item deferido é visível antes do `commit()`: `getItem()` o retorna como hit e
`hasItem()` é verdadeiro até ele expirar. `saveDeferred()` enfileira uma cópia e
inicia a expiração relativa na hora, então alterar o item depois não muda o valor
enfileirado, e um item que expira enquanto deferido não é gravado. `deleteItem()` e
`clear()` descartam itens enfileirados.

## Qual usar?

- Use **PSR-16** para cache chave/valor direto e o maior suporte de bibliotecas.
- Use **PSR-6** quando uma biblioteca exigir o modelo pool/item ou commits deferidos.
- Use a API nativa [`Cacheer`](./funcoes-cache.md) para escopos, `remember()`,
  `flexible()`, políticas, camadas e resiliência — recursos que os PSRs não cobrem.
