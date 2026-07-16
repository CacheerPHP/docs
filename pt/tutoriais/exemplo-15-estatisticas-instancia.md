# Exemplo 15 — Estatísticas e Gestão de Instâncias

*Novo na v5.0.0*

O CacheerPHP v5.0.0 adiciona métodos de diagnóstico e de ciclo de vida que facilitam inspecionar, testar e gerir instâncias de cache.

## stats() — Inspecionar o Estado do Cache

`stats()` devolve um array associativo com o driver ativo e o estado da compressão e da encriptação:

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useArrayDriver();
$cache->useCompression();

$info = $cache->stats();
print_r($info);
// [
//     'driver'      => 'Silviooosilva\CacheerPhp\CacheStore\ArrayCacheStore',
//     'compression' => true,
//     'encryption'  => false,
// ]
```

## getCacheStore() / setCacheStore() — Troca de Driver em Runtime

Pode obter ou substituir o store subjacente em runtime:

```php
use Silviooosilva\CacheerPhp\CacheStore\ArrayCacheStore;

// Obter o store atual
$store = $cache->getCacheStore();
echo get_class($store);

// Trocar para outro store em runtime
$cache->setCacheStore(new ArrayCacheStore('/tmp/cacheer.log'));
```

## resetInstance() — Limpar o Singleton Estático

Ao usar o CacheerPHP por chamadas estáticas, o singleton persiste durante a vida do processo. Use `resetInstance()` para o limpar — útil em testes ou workers de longa duração:

```php
// O uso estático cria um singleton
Cacheer::setDriver()->useArrayDriver();
Cacheer::putCache('key', 'value');

// Reset para começar do zero
Cacheer::resetInstance();

// A próxima chamada estática cria uma instância nova
Cacheer::setDriver()->useArrayDriver();
var_dump(Cacheer::has('key')); // false — os dados anteriores desapareceram
```

## setInstance() — Injetar uma Instância Personalizada

Substitua o singleton estático por uma instância pré-configurada. Útil para testes ou injeção de dependências:

```php
$testCache = new Cacheer();
$testCache->setDriver()->useArrayDriver();
$testCache->putCache('fixture', 'data');

// Injetar no singleton estático
Cacheer::setInstance($testCache);

// As chamadas estáticas passam a usar a instância injetada
echo Cacheer::getCache('fixture'); // "data"
```

## getOptions() / setOption() / setOptions()

Inspecione ou modifique a configuração em runtime:

```php
$cache = new Cacheer(['cacheDir' => __DIR__ . '/cache']);

// Ler todas as opções
$opts = $cache->getOptions();

// Definir uma única opção
$cache->setOption('expirationTime', '2 hours');

// Substituir todas as opções
$cache->setOptions([
    'cacheDir' => '/tmp/cache',
    'expirationTime' => '30 minutes',
]);
```
