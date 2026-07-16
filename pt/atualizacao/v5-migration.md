# Atualizar para a v5.0.0

Este guia cobre todas as alterações incompatíveis, as novas funcionalidades e os passos necessários para atualizar do CacheerPHP v4.x para a v5.0.0.

---

## Alteração de Requisitos

| | v4.x | v5.0.0 |
|-|------|--------|
| PHP | >= 8.0 | **>= 8.2** |
| psr/simple-cache | — | ^3.0 (novo) |
| psr/log | — | ^3.0 (novo) |

Atualize o seu `composer.json`:

```sh
composer require silviooosilva/cacheer-php:^5.0
```

---

## Alterações Incompatíveis

### 1. `$cacheStore` e `$options` agora são privados

O acesso direto às propriedades deixou de funcionar:

```php
// v4 (removido)
$cache->cacheStore;
$cache->options;
$cache->options['loggerPath'] = 'cacheer.log';

// v5 (use os accessors)
$cache->getCacheStore();
$cache->getOptions();
$cache->setOption('loggerPath', 'cacheer.log');
$cache->setOptions(['cacheDir' => '/tmp/cache']);
```

### 2. O valor de retorno de `add()` foi corrigido

Os valores de retorno estavam **invertidos** na v4. A v5 segue a convenção padrão:

```php
// v4 (errado)
$cache->add('existing_key', 'data');  // devolvia true
$cache->add('new_key', 'data');       // devolvia false

// v5 (correto)
$cache->add('existing_key', 'data');  // devolve false (a chave existe, nada foi escrito)
$cache->add('new_key', 'data');       // devolve true  (chave nova, valor guardado)
```

**Ação necessária:** Se o seu código verifica o retorno de `add()`, inverta a lógica.

### 3. `CACHE_FOREVER_TTL` mudou de `31536000000` para `PHP_INT_MAX`

O valor antigo (`31536000 * 1000`) podia causar overflow em sistemas de 32 bits. O novo valor é `PHP_INT_MAX`.

**Ação necessária:** Nenhuma para a maioria. Se compara TTLs com a constante antiga, atualize as comparações.

### 4. O formato de encriptação mudou (IV aleatório)

A v4 usava um IV determinístico derivado do hash da chave, fazendo com que payloads idênticos produzissem ciphertexts idênticos. A v5 gera um IV aleatório novo por escrita e coloca-o no início do ciphertext.

**Ação necessária:** Dados encriptados com a v4 **não podem** ser desencriptados pela v5. Limpe os caches encriptados antes de atualizar, ou reescreva os valores para os re-encriptar.

### 5. Formato de envelope do FileCacheStore

A v5 guarda o TTL por item num envelope JSON `{data, expires_at, ttl}` em vez de depender da data de modificação do ficheiro. A v5 inclui um fallback que lê ficheiros no formato da v4, pelo que os caches existentes continuam a funcionar até expirarem naturalmente.

**Ação necessária:** Nenhuma — o fallback trata disso. Para forçar o novo formato de imediato, limpe o cache de ficheiros depois de atualizar.

### 6. `CacheDataFormatter::toJson()` lança exceção em caso de falha

O tipo de retorno mudou de `string|false` para `string`, e o método agora usa `JSON_THROW_ON_ERROR`. Se a codificação falhar, é lançada uma `\JsonException` em vez de devolver `false`.

**Ação necessária:** Se verificava `false` no retorno de `toJson()`, passe a usar try/catch.

### 7. `Cacheer::getOptions()` não pode ser chamado estaticamente

Como `getOptions()` é agora um método público de instância em `Cacheer`, a resolução de métodos do PHP impede a chamada via `__callStatic`. Use o caminho `setConfig()`:

```php
// v4
Cacheer::getOptions();

// v5
Cacheer::setConfig()->getOptions();
// ou numa instância:
$cache->getOptions();
```

---

## Novas Funcionalidades

### Adaptador PSR-16 SimpleCache

```php
use Silviooosilva\CacheerPhp\Psr\Psr16CacheAdapter;

$psr = new Psr16CacheAdapter($cache);
$psr->set('key', 'value', 3600);
$psr->get('key', 'default');
$psr->getMultiple(['a', 'b', 'c']);
```

Ver a [referência do Adaptador PSR-16](../api/psr16-adapter.md).

### Logger PSR-3

O `CacheLogger` agora estende `\Psr\Log\AbstractLogger`, ficando compatível com qualquer consumidor PSR-3.

### TTL com DateInterval

Todos os métodos que aceitam TTL agora também aceitam `\DateInterval` e `null`:

```php
$cache->putCache('key', 'data', ttl: new \DateInterval('PT30M'));
$cache->putCache('key', 'data', ttl: null);  // para sempre
```

### Cache de Valores Falsy

`0`, `0.0`, `''`, `'0'`, `false` e `[]` são agora hits de cache válidos. A biblioteca usa `isSuccess()` internamente em vez de `!empty()`, pelo que `remember()`, `increment()` e `getAndForget()` funcionam corretamente com estes valores.

### Gestão de Instâncias

```php
$cache->stats();              // ['driver' => '...', 'compression' => bool, 'encryption' => bool]
Cacheer::resetInstance();     // limpa o singleton estático
Cacheer::setInstance($cache); // substitui o singleton estático
```

### Novos Accessors

```php
$cache->getCacheStore();             // devolve o driver CacheerInterface ativo
$cache->setCacheStore($newDriver);   // troca o driver em runtime
$cache->getOptions();                // devolve o array de opções
$cache->setOption('key', 'value');   // define uma única opção
$cache->setOptions([...]);           // substitui todas as opções
```

---

## Checklist de Migração

1. Atualize o PHP para 8.2+
2. Execute `composer require silviooosilva/cacheer-php:^5.0`
3. Procure `->cacheStore` e `->options` no código — substitua pelos métodos accessor
4. Procure chamadas a `->add()` — verifique se a lógica do retorno está correta para a nova semântica
5. Se usa encriptação, limpe os caches encriptados antes de fazer deploy
6. Se chama `Cacheer::getOptions()` estaticamente, mude para `Cacheer::setConfig()->getOptions()`
7. Se verifica `toJson() === false`, mude para try/catch de `\JsonException`
8. Corra a sua suite de testes: `vendor/bin/phpunit`
