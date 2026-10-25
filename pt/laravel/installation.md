# Instalar o CacheerPHP para Laravel

Esta página adiciona uma store CacheerPHP a uma aplicação Laravel 12 ou 13 e torna-a o cache predefinido. Para voltar atrás, basta alterar uma linha no `.env`.

## 1. Instalar o pacote

Na raiz da aplicação:

```sh
composer require cacheerphp/laravel
```

O Laravel deteta o service provider automaticamente. O pacote instala o CacheerPHP 6 como dependência.

## 2. Adicionar uma store

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

## 3. Torná-la a predefinida

No `.env`:

```sh
CACHE_STORE=cacheer
```

Execute `php artisan config:clear` depois da alteração. Nos deployments que
guardam a configuração em cache, volte a gerá-la com `php artisan config:cache`.

Um `CACHE_STORE=database` existente prevalece sobre o valor de recurso em
`env('CACHE_STORE', 'cacheer')`; altere a variável de ambiente explicitamente.

Tudo o que usa a store de cache predefinida passa a correr sobre o CacheerPHP: `Cache::`, `cache()`, locks, jobs únicos, o scheduler e os rate limiters. Para a usar apenas em algumas chamadas, mantenha a predefinição e chame `Cache::store('cacheer')`.

## 4. Criar as tabelas (apenas backend de base de dados)

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

## 5. Confirmar que funciona

```sh
php artisan tinker
```

```php
Cache::put('hello', 'world', 60);
Cache::get('hello');          // "world"
Cache::getStore()::class;     // "Cacheer\Laravel\CacheerStore"
```

Para acompanhar o tráfego, instale o [Cacheer Monitor](../cacheer-monitor/quick-start.md); todas as stores `cacheer` reportam para ele sem mais configuração.
Os eventos de failover requerem CacheerPHP 6.1 e uma versão compatível do Monitor.
Os eventos normais de cache também funcionam com CacheerPHP 6.0.

## Voltar atrás

Defina `CACHE_STORE` com a store anterior. O CacheerPHP e o Laravel usam formatos de armazenamento diferentes, por isso a outra store começa sem as entradas escritas pelo CacheerPHP; os valores são recalculados à medida que são pedidos.
