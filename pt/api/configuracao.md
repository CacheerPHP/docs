# Referência da API — Configuração

Sempre defina o driver a ser usado primeiro e, só então, defina as configurações usando **setConfig()**.

Veja também:
[Selecionar driver (setDriver)](./drivers.md)

#### `setConfig()`

```php

<?php

require_once __DIR__ . "/../vendor/autoload.php"; 

$Cacheer = new Cacheer();
$Cacheer->setConfig();
```

```php
Cacheer::setConfig();
```

> Observação: Métodos de configuração também podem ser chamados estaticamente, por exemplo: `Cacheer::setConfig()->setDatabaseConnection('mysql');`

Configura o banco de dados para armazenar o cache.
```php

<?php

require_once __DIR__ . "/../vendor/autoload.php"; 

$Cacheer = new Cacheer();
$Cacheer->setConfig()->setDatabaseConnection(string $driver)
```

```php
Cacheer::setConfig()->setDatabaseConnection(string $driver);
```

- Parâmetros:

```php
$driver: Driver do banco. Valores possíveis: 'mysql', 'pgsql', 'sqlite'.
```

**Exemplo:**

```php

<?php

require_once __DIR__ . "/../vendor/autoload.php"; 

$Cacheer = new Cacheer();
$Cacheer->setConfig()->setDatabaseConnection('mysql');
```

```php
Cacheer::setConfig()->setDatabaseConnection('mysql');
```

Também é possível definir o driver no arquivo `.env` através da variável `DB_CONNECTION`, com os mesmos valores.

Timezone
---

```php

<?php

require_once __DIR__ . "/../vendor/autoload.php"; 

$Cacheer = new Cacheer();
$Cacheer->setConfig()->setTimeZone(string $timezone);
```

```php
Cacheer::setConfig()->setTimeZone(string $timezone);
```

Define o fuso horário para as operações de cache.
- Parâmetros

```php
$timezone: Timezone no formato PHP (ex.: 'UTC', 'America/Sao_Paulo').
```

**Exemplo:**

```php
$Cacheer->setConfig()->setTimeZone('UTC');
```

```php
Cacheer::setConfig()->setTimeZone('UTC');
```

Consulte os timezones suportados pelo PHP:
https://www.php.net/manual/pt_BR/timezones.php

Logger
---

```php
$Cacheer->setConfig()->setLoggerPath(string $path);
```

```php
Cacheer::setConfig()->setLoggerPath(string $path);
```
Define o caminho onde os logs serão armazenados.

- Parâmetros

```php
$path: Caminho completo para o arquivo de logs.
```

**Exemplo:**

```php
$Cacheer->setConfig()->setLoggerPath('/caminho/para/logs/CacheerPHP.log');
```

```php
Cacheer::setConfig()->setLoggerPath('/caminho/para/logs/CacheerPHP.log');
```

