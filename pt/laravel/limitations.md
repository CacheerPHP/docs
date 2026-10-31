# Diferenças e limitações

Uma store `cacheer` segue os contratos de cache do Laravel, mas alguns comportamentos diferem das stores nativas do Laravel. Cada um é intencional; esta página explica porquê e o que fazer em alternativa.

## Sem `Cache::tags()`

As tags do Laravel e as do CacheerPHP significam coisas diferentes. Um cache com tags do Laravel separa as chaves pelo conjunto de tags, por isso `Cache::tags(['users'])->get('a')` e `Cache::get('a')` são entradas diferentes. Uma tag do CacheerPHP é um conjunto de invalidação associado a uma chave existente. Mapear uma na outra quebraria a semântica de ambas, por isso `Cache::tags()` lança `BadMethodCallException` numa store `cacheer`.

Para invalidar grupos de chaves, use as tags ou os escopos do CacheerPHP através do contrato injetado; consulte [funcionalidades do Laravel](./framework-features.md#funcionalidades-do-cacheerphp-alm-de-cache).

## Os locks custam mais viagens de ida e volta

O Laravel precisa de donos de lock que outro processo possa libertar (`restoreLock()`, `forceRelease()`). Os locks do CacheerPHP não expõem o dono, por isso um lock do Laravel é um registo de cache com o dono, alterado sob um lock curto do CacheerPHP. Adquirir um lock no Redis custa quatro viagens de ida e volta, onde o lock Redis do Laravel custa uma. Em troca, a libertação é segura quanto ao dono em todos os backends, e os locks recusam por segurança durante uma falha.

Se um trecho de código adquire muitos locks por pedido, meça antes de mudar.

## Os contadores são números inteiros

`Cache::increment('visits', 1.5)` lança `InvalidArgumentException` em vez de truncar o valor, e incrementar um valor que não seja inteiro falha sem o alterar. Isso inclui uma string numérica, como `'5'` guardado com `Cache::put()`, que as stores Redis e de base de dados do Laravel incrementariam. Guarde os contadores como inteiros, ou crie-os com `increment()`.

## Contadores e locks durante uma falha

Enquanto o backend principal está em baixo, os seus contadores e locks ficam indisponíveis, mesmo com um [fallback](./configuration.md#failover): um contador ou lock guardado no fallback não seria partilhado com outros servidores.

- Os locks não são adquiridos: `get()` devolve `false`.
- `Cache::add()`, `increment()` e `decrement()` lançam `StoreOperationFailedException`.

Os rate limiters do Laravel usam `add()` e `increment()`, por isso as rotas com o middleware `throttle` falham até o principal recuperar. Se tiverem de continuar a funcionar, aponte `cache.limiter` em `config/cache.php` para uma store que não dependa do mesmo backend.

## Chaves

As chaves têm no máximo 1024 bytes e não podem conter caracteres de controlo. Chaves maiores ou inválidas lançam `InvalidKeyException`.

## Ligações Redis

O backend redis abre a sua própria ligação com os dados da ligação do Laravel que indicar. Ignora as opções `prefix`, `serializer` e `compression` do redis do Laravel, e clusters Redis não são suportados. Consulte a [configuração](./configuration.md#backends).

## Transações de base de dados

O backend de base de dados usa a ligação que indicar em `connection` e acompanha-a quando o Laravel volta a ligar-se. As escritas no cache e os registos de lock feitos dentro de `DB::transaction()` nessa ligação pertencem à transação: são revertidos com ela, e os outros processos só os veem depois do commit. Para manter o cache independente das suas transações, defina uma segunda ligação para a mesma base de dados e indique-a em `connection`.

## O failover é por processo

Um processo que termina durante uma falha esquece as chaves que alterou, por isso, com PHP-FPM, um valor alterado ou apagado durante uma falha do Redis pode voltar a ser servido depois de o Redis recuperar, até expirar. Consulte a [configuração](./configuration.md#failover) para ver como mitigar.

## `null` em cache

O repositório do Laravel trata `null` como um miss, seja qual for a store. `Cache::put('key', null)` seguido de `Cache::get('key', 'default')` devolve `'default'`. O contrato do CacheerPHP injetado distingue um `null` guardado de um miss, através de `entry()`.

## Atualizar a partir de silviooosilva/cacheerphp-laravel 0.1

O pacote 0.1 envolvia o CacheerPHP 4 e 5 numa facade `Cacheer` própria. O `cacheerphp/laravel` substitui-o por um driver de cache:

- Substitua `Cacheer::putCache()` e `Cacheer::getCache()` por `Cache::put()` e `Cache::get()`, ou pelo contrato do CacheerPHP injetado.
- Passe as definições de `config/cacheer.php` para uma store `cacheer` em `config/cache.php`.
- Os comandos `cacheer:monitor`, `cacheer:events-clear` e `cacheer:diagnostics` deixaram de existir. Use o [Cacheer Monitor](../cacheer-monitor/index.md) e a [CLI `cacheer`](../guias/cli.md) incluída no CacheerPHP.
- O CacheerPHP 6 não lê dados escritos pelo CacheerPHP 5. Comece com um cache vazio, por exemplo com um novo `prefix`. Consulte a [migração a partir da v5](../atualizacao/v5-migration.md).
