# Primeiros Passos

O CacheerPHP 6 é um cache baseado em instâncias: um núcleo `Cacheer` enxuto sobre um
contrato `Store` mínimo, com capacidades opcionais (lotes, tags, locks, contadores
atômicos), decorators componíveis (camadas, resiliente, instrumentado), um pipeline
de armazenamento autenticado e adaptadores PSR-16/PSR-6.

## Requisitos

| Requisito | Detalhes |
|-----------|----------|
| **PHP** | 8.3 ou mais recente |
| **Opcional** | `ext-openssl` (criptografia AES-256-GCM), `ext-zlib` (gzip) |
| **Opcional** | `ext-pdo` + driver PDO para a store de banco de dados |
| **Opcional** | `predis/predis` ou `ext-redis` para a store Redis |

O núcleo instala e funciona com as stores Array e File **sem** nenhuma das partes
opcionais.

## Instalação

```sh
composer require silviooosilva/cacheer-php
```

Isso já inclui os contratos PSR (`psr/simple-cache`, `psr/cache`, `psr/log`,
`psr/event-dispatcher`).

## Início Rápido

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

// Em memória, sem dependências (ótimo para testes e execuções curtas de CLI).
$cache = Cacheer::inMemory();

// Grave com um TTL: segundos, "10 minutes", um DateInterval ou null (para sempre).
$cache->set('user:123', ['name' => 'John Doe'], ttl: '10 minutes');

// Leia (retorna o padrão em caso de miss).
$user = $cache->get('user:123', default: null);

// Calcule uma única vez: executa o callback no miss, guarda e retorna.
$report = $cache->remember('report:daily', ttl: 3600, callback: fn () => build_report());

// Existência e remoção.
$cache->has('user:123');   // true
$cache->delete('user:123');
```

## Escolhendo uma store

Troque a store, mantenha a API. Veja [Stores e capacidades](../api/drivers.md).

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::inMemory();                    // store de array em memória
$cache = Cacheer::file('/var/cache/app');        // persistente, sem dependências
$cache = Cacheer::database($pdo, 'cacheer');     // injete seu próprio PDO
$cache = Cacheer::redis($connection);            // adaptador predis ou phpredis
```

Crie o schema do banco explicitamente antes (nunca como efeito colateral):

```php
use Silviooosilva\CacheerPhp\Stores\Support\DatabaseStoreSchema;

DatabaseStoreSchema::migrate($pdo, 'cacheer');
```

## Escopos

Escopos substituem namespaces por string por keyspaces isolados que você limpa
separadamente. Veja o [guia de Escopos](../guias/escopos.md).

```php
$cache->scope('reports')->set('daily', $rows);
$cache->scope('billing')->set('daily', $invoice); // entrada independente
$cache->scope('reports')->clear();                // limpa apenas esse escopo
```

## Cálculo único e stale-while-revalidate

```php
// Executa o callback uma vez mesmo sob estampede concorrente.
$user = $cache->remember('user:42', '10 minutes', fn () => $users->find(42));

// Serve fresco por 30s, depois serve stale enquanto um worker recarrega, até 300s.
$feed = $cache->flexible('feed', fresh: 30, stale: 300, callback: fn () => build_feed());
```

Veja [Remember e locks](../guias/remember-e-locks.md) e
[Stale-while-revalidate](../guias/stale-while-revalidate.md).

## Formatos de TTL

Um TTL pode ser int (segundos), string legível, `DateInterval` ou `null` (para
sempre). Veja [TTL e Clock](../api/construtor-de-tempo.md).

```php
$cache->set('key', $data, ttl: 3600);
$cache->set('key', $data, ttl: '2 hours');
$cache->set('key', $data, ttl: new \DateInterval('PT30M'));
$cache->set('key', $data, ttl: null); // para sempre
```

## Adaptadores PSR

```php
use Silviooosilva\CacheerPhp\Psr\{Psr16Cache, Psr6Pool};
use Silviooosilva\CacheerPhp\Support\SystemClock;

$psr16 = new Psr16Cache($cache);                    // Psr\SimpleCache\CacheInterface
$pool  = new Psr6Pool($cache, new SystemClock());   // Psr\Cache\CacheItemPoolInterface
```

Veja [Adaptadores PSR-16 e PSR-6](../api/psr16-adapter.md).

## Vindo da v5?

A migração é quase toda mecânica — renomeie os métodos v5 para os nomes v6
(`putCache`→`set`, `getCache`→`get`, namespace posicional → `scope()`), e deixe
seus dados já cacheados se atualizarem sozinhos via reescrita na leitura. Um
conjunto Rector opcional automatiza as renomeações comuns; se um serviço ainda não
puder migrar, mantenha-o em `^5.2`.

Veja o [guia de atualização](../atualizacao/index.md) para o mapeamento completo.

## Próximos Passos

- [Guias](../guias/configuracao.md) — aprofundamentos por funcionalidade
- [Referência da API](../api/index.md) — documentação completa dos métodos
- [Tutoriais](../tutoriais/index.md) — exemplos objetivos
- [Guia de atualização](../atualizacao/index.md) — atualizando da v5
