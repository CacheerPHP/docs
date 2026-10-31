# Instalar o CacheerPHP para Laravel

Esta página instala o CacheerPHP para Laravel numa aplicação Laravel 12 ou 13, mostra como correm as suas stores de cache e, se quiser, adiciona uma store sobre o armazenamento próprio do CacheerPHP.

## 1. Instalar o pacote

Na raiz da aplicação:

```sh
composer require cacheerphp/laravel
```

O Laravel deteta o service provider automaticamente. O pacote instala o CacheerPHP 6.1.1 ou posterior como dependência.

## 2. Ver as suas stores

As stores que já tem passam a correr através do CacheerPHP, sem configuração. Para ver quais:

```sh
php artisan cacheer:doctor
```

```text
+----------+----------+------------------------------------------------------------------------+
| Store    | Driver   | CacheerPHP                                                             |
+----------+----------+------------------------------------------------------------------------+
| array    | array    | not routed: it lacks add() or locks, which routing must mirror exactly |
| database | database | routed                                                                 |
| file     | file     | routed                                                                 |
| redis    | redis    | routed                                                                 |
| failover | failover | not routed: it wraps other stores, which are routed themselves         |
+----------+----------+------------------------------------------------------------------------+
```

Uma store encaminhada devolve o que devolvia antes e mantém os seus dados. Uma store que não é encaminhada fica exatamente como estava. Consulte a [configuração](./configuration.md#encaminhar-as-stores-existentes) para ver porque algumas stores ficam nativas e como desligar o encaminhamento.

Para acompanhar o tráfego, instale o [Cacheer Monitor](../cacheer-monitor/quick-start.md): todas as stores encaminhadas e todas as stores `cacheer` reportam para ele sem mais configuração.

Se é tudo o que precisa, terminou. Os passos seguintes adicionam uma store sobre o armazenamento próprio do CacheerPHP, para failover, cache em camadas ou criptografia.

## 3. Adicionar uma store `cacheer`

Em `config/cache.php`, adicione uma store que use o driver `cacheer`:

```php
'stores' => [

    'cacheer' => [
        'driver' => 'cacheer',
        'backend' => 'file',
    ],

    // ...as stores que já tem
],
```

O backend `file` não precisa de serviços e guarda as entradas em `storage/framework/cache/cacheer`. Para Redis, base de dados, failover ou criptografia, consulte a [configuração](./configuration.md).

## 4. Torná-la a predefinida

No `.env`:

```sh
CACHE_STORE=cacheer
```

Execute `php artisan config:clear` depois da alteração. Nos deployments que
guardam a configuração em cache, volte a gerá-la com `php artisan config:cache`.

Um `CACHE_STORE=database` existente prevalece sobre o valor de recurso em
`env('CACHE_STORE', 'cacheer')`; altere a variável de ambiente explicitamente.

Tudo o que usa a store de cache predefinida passa a correr sobre o CacheerPHP: `Cache::`, `cache()`, locks, jobs únicos, o scheduler e os rate limiters. Para a usar apenas em algumas chamadas, mantenha a predefinição e chame `Cache::store('cacheer')`.

## 5. Criar as tabelas (apenas backend de base de dados)

Ignore este passo, a menos que uma store `cacheer` use `'backend' => 'database'`, como store principal ou como fallback. Publique a migration:

```sh
php artisan vendor:publish --tag=cacheer-migrations
```

Antes de a executar, edite a lista `$tables` da migration publicada para
corresponder aos seus backends de base de dados, incluindo os fallbacks. O valor
predefinido é `[[null, 'cacheer_store']]`, na ligação de base de dados predefinida
do Laravel. Por exemplo:

```php
private array $tables = [
    ['mysql', 'cacheer_store'],
    ['mysql', 'cacheer_fallback'],
];
```

Inclua cada par de ligação/tabela uma só vez e execute `php artisan migrate`.
Mantenha a lista fixa depois disso, para que o rollback reverta a migration
original mesmo que `config/cache.php` mude. Use uma nova migration ao adicionar
outro backend de base de dados.

## 6. Confirmar que funciona

```sh
php artisan tinker
```

```php
Cache::put('hello', 'world', 60);
Cache::get('hello');          // "world"
Cache::getStore()::class;     // "Cacheer\Laravel\CacheerStore"
```

## Voltar atrás

Para deixar de encaminhar, remova o pacote ou [desligue o encaminhamento](./configuration.md#encaminhar-as-stores-existentes). As stores encaminhadas mantêm o formato do próprio Laravel, por isso não há nada a migrar.

Para deixar de usar uma store `cacheer`, defina `CACHE_STORE` com a store anterior. O CacheerPHP e o Laravel usam formatos de armazenamento diferentes, por isso a outra store começa sem as entradas escritas pelo CacheerPHP; os valores são recalculados à medida que são pedidos.
