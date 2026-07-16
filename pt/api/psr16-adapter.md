# Adaptador PSR-16 SimpleCache

*Novo na v5.0.0*

O `Psr16CacheAdapter` envolve qualquer instância `Cacheer` e expõe a interface padrão `\Psr\SimpleCache\CacheInterface`. Isto permite usar o CacheerPHP em qualquer lugar onde se espera um cache PSR-16 — containers de serviços de frameworks, bibliotecas de terceiros ou as suas próprias dependências tipadas.

## Instalação

O adaptador está incluído no pacote principal. A dependência `psr/simple-cache` ^3.0 é instalada automaticamente.

```sh
composer require silviooosilva/cacheer-php
```

## Uso Básico

```php
<?php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Psr\Psr16CacheAdapter;

$cacheer = new Cacheer(['cacheDir' => __DIR__ . '/cache']);
$cache   = new Psr16CacheAdapter($cacheer);

// set / get / has / delete
$cache->set('user:1', ['name' => 'Alice'], 3600);
$cache->get('user:1');               // ['name' => 'Alice']
$cache->get('missing', 'default');   // 'default'
$cache->has('user:1');               // true
$cache->delete('user:1');
```

## Construtor

```php
new Psr16CacheAdapter(Cacheer $cache, string $namespace = '')
```

| Parâmetro | Descrição |
|-----------|-----------|
| `$cache` | A instância Cacheer subjacente a que o adaptador delega |
| `$namespace` | Namespace opcional aplicado a todas as chaves (isola instâncias diferentes do adaptador) |

## Métodos

### Operações de Item Único

| Método | Assinatura | Descrição |
|--------|-----------|-----------|
| `get` | `get(string $key, mixed $default = null): mixed` | Obtém um valor; devolve `$default` em caso de miss |
| `set` | `set(string $key, mixed $value, int\|DateInterval\|null $ttl = null): bool` | Guarda um valor |
| `delete` | `delete(string $key): bool` | Remove uma chave |
| `has` | `has(string $key): bool` | Verifica a existência |
| `clear` | `clear(): bool` | Limpa todo o cache |

### Operações em Lote

| Método | Assinatura | Descrição |
|--------|-----------|-----------|
| `getMultiple` | `getMultiple(iterable $keys, mixed $default = null): iterable` | Obtém várias chaves de uma vez |
| `setMultiple` | `setMultiple(iterable $values, int\|DateInterval\|null $ttl = null): bool` | Guarda vários pares chave/valor |
| `deleteMultiple` | `deleteMultiple(iterable $keys): bool` | Remove várias chaves |

## Comportamento do TTL

| Entrada | Efeito |
|---------|--------|
| `null` | Guarda para sempre (`PHP_INT_MAX` segundos) |
| `int` positivo | Segundos até expirar |
| `0` ou `int` negativo | Expira imediatamente (a chave é apagada) |
| `\DateInterval` | Convertido automaticamente para segundos |

```php
$cache->set('forever', 'data', null);                      // sem expiração
$cache->set('timed', 'data', 1800);                        // 30 minutos
$cache->set('interval', 'data', new DateInterval('PT1H')); // 1 hora
$cache->set('ephemeral', 'data', 0);                       // apagado imediatamente
```

## Validação de Chaves

A PSR-16 define caracteres reservados que **não** podem aparecer nas chaves. O adaptador lança `CacheInvalidArgumentException` (que implementa `\Psr\SimpleCache\InvalidArgumentException`) para:

- Strings vazias
- Chaves que contenham qualquer um de: `{ } ( ) / \ @ :`

```php
use Silviooosilva\CacheerPhp\Exceptions\CacheInvalidArgumentException;

try {
    $cache->get('bad:key');
} catch (CacheInvalidArgumentException $e) {
    // "Cache key "bad:key" contains reserved PSR-16 characters"
}
```

## Isolamento por Namespace

Crie vários adaptadores com namespaces diferentes para isolar módulos que partilham a mesma instância Cacheer:

```php
$users    = new Psr16CacheAdapter($cacheer, 'users');
$sessions = new Psr16CacheAdapter($cacheer, 'sessions');

$users->set('config', 'user-value');
$sessions->set('config', 'session-value');

$users->get('config');    // 'user-value'
$sessions->get('config'); // 'session-value'
```

## Exemplo de Integração com Framework

Registe o adaptador num container ao estilo Laravel:

```php
$container->singleton(\Psr\SimpleCache\CacheInterface::class, function () {
    $cacheer = new Cacheer(['cacheDir' => storage_path('cache')]);
    return new Psr16CacheAdapter($cacheer);
});
```
