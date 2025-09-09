# Referência da API — Construtor de Opções (OptionBuilder)

O **OptionBuilder** permite definir diferentes parâmetros de configuração do CacheerPHP de forma mais segura e fluente, evitando erros de digitação.

Veja também o **Construtor de Tempo (TimeBuilder)**: [Introdução ao TimeBuilder](./construtor-de-tempo.md)

Há builders para File, Redis e Database, ajudando a definir opções de forma fluente e segura.

Você já deve ter percebido que parâmetros são suscetíveis a erros de digitação, certo?
O **OptionBuilder** nasce para eliminar esses erros possíveis.

#### `setUp(array $options): void`

Inicializa a instância de `Cacheer` com as opções informadas.

- Parâmetros:
  - `array $options`: Array associativo com opções de configuração (ex.: `driver`, `path`, etc.).

- Exemplo:
  ```php
  $cache = new Cacheer();
  $options = [
      'driver' => 'file',
      'path' => '/tmp/cache',
  ];
  $cache->setUp($options);
  ```

#### `getOptions(): array`

Retorna as opções atuais de configuração da instância `Cacheer`.

- Retorno:
  - `array`: Opções atuais.

- Exemplo:
  ```php
  $cache = new Cacheer();
  $options = $cache->getOptions();
  var_dump($options);
  ```

#### `OptionBuilder()`

O **OptionBuilder** possui métodos específicos para configurar cada tipo de driver suportado. Cada um inicializa a configuração para um driver e retorna uma instância do builder correspondente.

`forFile()`

```php
<?php
$Options = OptionBuilder::forFile();
```
Inicializa o FileCacheStore, permitindo configurar diretório de cache, tempo de expiração e limpeza periódica.

Métodos após `forFile()`

```
dir(string $path) → Define o diretório onde os arquivos de cache serão armazenados.
expirationTime(string $time) → Define o tempo de expiração dos arquivos no cache.
flushAfter(string $interval) → Define um intervalo para executar limpeza automática do cache.
build() → Finaliza a configuração e retorna um array de opções pronto para uso.
```

Exemplo de uso

```php
<?php
require_once __DIR__ . "/../vendor/autoload.php"; 

$Options = OptionBuilder::forFile()
    ->dir(__DIR__ . "/cache")
    ->expirationTime("2 hours")
    ->flushAfter("1 day")
    ->build();

$Cacheer = new Cacheer($Options);
$Cacheer->setDriver()->useFileDriver(); // File Driver
```

> Observação: Métodos do Cacheer também podem ser chamados estaticamente, por exemplo: `Cacheer::setDriver()->useFileDriver();`

`forRedis()`

```php
<?php
$Options = OptionBuilder::forRedis()
    ->setNamespace('app:')
    ->expirationTime('2 hours')
    ->flushAfter('1 day')
    ->build();
```
Inicializa opções do Redis. É possível definir um prefixo de namespace para as chaves e, opcionalmente, TTL padrão e intervalo de auto-flush.

Comportamento:
- `expirationTime` define um TTL padrão usado quando você não informa TTL no `putCache()` ou quando usa o padrão implícito de `3600`. TTLs explícitos diferentes de `3600` sempre prevalecem.
- `flushAfter` habilita uma verificação de auto-flush na inicialização do store. Se o último flush for mais antigo que o intervalo, o Cacheer executa `flushCache()` para o namespace do Redis.

Métodos após `forRedis()`

```
setNamespace(string $prefix) → Define o prefixo de namespace das chaves.
expirationTime(string $time) → Sinaliza um TTL padrão.
flushAfter(string $interval) → Sinaliza um intervalo de auto-flush.
build() → Finaliza e retorna as opções.
```

`forDatabase()`

```php

<?php
$Options = OptionBuilder::forDatabase()
    ->table('cache_items')
    ->expirationTime('1 day')
    ->flushAfter('7 days')
    ->build();
```
Inicializa opções de Banco de Dados. Você pode definir a tabela de armazenamento e controles de tempo opcionais.

Comportamento:
- `expirationTime` define um TTL padrão usado quando você não informa TTL no `putCache()` ou quando usa o padrão implícito de `3600`. TTLs explícitos diferentes de `3600` prevalecem.
- `flushAfter` habilita uma verificação de auto-flush na inicialização; se o último flush for mais antigo que o intervalo, o Cacheer executa `flushCache()` para a tabela configurada.

Métodos após `forDatabase()`

```
table(string $table) → Define a tabela de armazenamento.
expirationTime(string $time) → Sinaliza um TTL padrão.
flushAfter(string $interval) → Sinaliza um intervalo de auto-flush.
build() → Finaliza e retorna as opções.
```

O **OptionBuilder** simplifica a configuração do **CacheerPHP**, eliminando erros de digitação e tornando o processo mais intuitivo. 🚀

Exemplos
---

Redis com TTL padrão e auto-flush:

```php

<?php
$options = OptionBuilder::forRedis()
  ->setNamespace('app:')
  ->expirationTime('2 hours')
  ->flushAfter('1 day')
  ->build();

$cache = new Cacheer($options);
$cache->setDriver()->useRedisDriver();

// Usa TTL padrão (2 horas)
$cache->putCache('session_123', ['id' => 123]);

// TTL explícito prevalece (10 minutos)
$cache->putCache('session_456', ['id' => 456], '', '10 minutes');
```

Banco de dados com tabela customizada, TTL padrão e auto-flush:

```php

<?php
$options = OptionBuilder::forDatabase()
  ->table('cache_items')
  ->expirationTime('30 minutes')
  ->flushAfter('7 days')
  ->build();

$cache = new Cacheer($options);
$cache->setDriver()->useDatabaseDriver();

// Usa TTL padrão (30 minutos)
$cache->putCache('user_1', ['name' => 'Jane']);
```

