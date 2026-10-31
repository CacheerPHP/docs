# Registar o CacheerPHP nos serviços Laravel

Registe o CacheerPHP no service container e injete o contrato `Cache` nos serviços. Este guia usa Laravel 13.x e CacheerPHP 6.x. O binding adiciona um cache para os seus serviços; a configuração de cache existente do Laravel continua a controlar as funcionalidades do framework.

Para que as próprias funcionalidades de cache do Laravel (`Cache::`, locks, filas, scheduler) corram sobre o CacheerPHP, use o [CacheerPHP para Laravel](../laravel/index.md), um driver de cache. Este guia é para manter o cache do Laravel como está.

Se está a decidir se deve adicionar CacheerPHP, [compare as duas APIs](./cacheerphp-vs-laravel-cache.md). Para um exemplo independente e pronto a executar, consulte os [primeiros passos](../primeiros-passos/index.md).

## 1. Instalar

Na raiz da aplicação Laravel:

```sh
composer require silviooosilva/cacheer-php:"^6.0"
```

## 2. Registar o cache

Adicione este binding ao método `register()` existente em `app/Providers/AppServiceProvider.php`, preservando os restantes bindings:

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Contracts\Cache;

public function register(): void
{
    $this->app->scoped(Cache::class, static fn (): Cache => Cacheer::build()
        ->file(storage_path('framework/cache/cacheer-v6'))
        ->create());
}
```

A store de ficheiros persiste entre pedidos num servidor. O lifetime `scoped()` fornece uma instância por pedido ou job nos ciclos de workers suportados pelo Laravel. Veja os [bindings scoped do Laravel](https://laravel.com/docs/13.x/container#binding-scoped-singletons). Mantenha o diretório com permissão de escrita e fora de `public/`. Para vários servidores, escolha Redis ou uma store de base de dados partilhada no mesmo binding.

## 3. Injetar num serviço

Para um catálogo público, crie `app/Services/ProductCatalog.php`. O exemplo pressupõe um modelo `App\Models\Product` existente com colunas `id`, `name` e `price`:

```php
<?php

namespace App\Services;

use App\Models\Product;
use Silviooosilva\CacheerPhp\Contracts\Cache;

final class ProductCatalog
{
    public function __construct(private readonly Cache $cache)
    {
    }

    public function all(): array
    {
        return $this->cache->scope('catalog')->remember(
            'products',
            300,
            static fn (): array => Product::query()
                ->get(['id', 'name', 'price'])
                ->toArray(),
        );
    }

    public function invalidate(): void
    {
        $this->cache->scope('catalog')->delete('products');
    }
}
```

Injete `ProductCatalog` no controlador ou experimente uma rota em `routes/web.php`:

```php
use App\Services\ProductCatalog;
use Illuminate\Support\Facades\Route;

Route::get('/catalog', static fn (ProductCatalog $catalog): array => $catalog->all());
```

O primeiro pedido consulta a base de dados; os seguintes reutilizam o resultado durante cinco minutos. Chame `$catalog->invalidate()` depois de atualizar um produto com sucesso e após o commit da transação. O próximo pedido volta a carregar o catálogo.

O exemplo serve dados públicos partilhados. Para dados de um tenant, resolva a autorização antes de ler o cache e inclua o tenant autorizado e os filtros da consulta no escopo/chave.

## 4. Confirmar a integração

Consulte `/catalog` duas vezes e inspecione as queries ou o [Cacheer Monitor](../cacheer-monitor/quick-start.md). Num teste, injete `Cacheer::inMemory()` para armazenamento local rápido. O mesmo serviço `ProductCatalog` continua a depender do contrato nativo de cache.

Para stale-while-revalidate, configure um [executor diferido](./stale-while-revalidate.md) para o ciclo Laravel: o executor predefinido do CacheerPHP é síncrono. Para mais configuração, veja [stores e capacidades](../api/drivers.md), [políticas](./politicas.md) e [observabilidade](./observabilidade.md).
