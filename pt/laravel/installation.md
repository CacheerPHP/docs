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

Tudo o que usa a store de cache predefinida passa a correr sobre o CacheerPHP: `Cache::`, `cache()`, locks, jobs únicos, o scheduler e os rate limiters. Para a usar apenas em algumas chamadas, mantenha a predefinição e chame `Cache::store('cacheer')`.

## 4. Criar as tabelas (apenas backend de base de dados)

Ignore este passo, a menos que uma store `cacheer` use `'backend' => 'database'`, como store principal ou como fallback. Publique a migration e execute-a:

```sh
php artisan vendor:publish --tag=cacheer-migrations
php artisan migrate
```

A migration cria as tabelas de todas as stores `cacheer` configuradas com o backend de base de dados, na ligação de cada store.

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

## Voltar atrás

Defina `CACHE_STORE` com a store anterior. O CacheerPHP e o Laravel usam formatos de armazenamento diferentes, por isso a outra store começa sem as entradas escritas pelo CacheerPHP; os valores são recalculados à medida que são pedidos.
