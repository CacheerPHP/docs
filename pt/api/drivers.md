# Referência da API — Drivers

```php
<?php

require_once __DIR__ . "/../vendor/autoload.php"; 

$Cacheer = new Cacheer();
$Cacheer->setDriver();
```

```php
Cacheer::setDriver();
```

> Observação: Os métodos de driver também podem ser chamados estaticamente, por exemplo: `Cacheer::setDriver()->useFileDriver();`

Definir driver baseado em arquivos:
```php
<?php

require_once __DIR__ . "/../vendor/autoload.php"; 

$Cacheer = new Cacheer();
$Cacheer->setDriver()->useFileDriver();
```

```php
Cacheer::setDriver()->useFileDriver();
```

Definir driver baseado em banco de dados:
```php
<?php

require_once __DIR__ . "/../vendor/autoload.php"; 

$Cacheer = new Cacheer();
$Cacheer->setDriver()->useDatabaseDriver();
```

```php
Cacheer::setDriver()->useDatabaseDriver();
```

Definir driver baseado em Redis:
```php
<?php

require_once __DIR__ . "/../vendor/autoload.php"; 

$Cacheer = new Cacheer();
$Cacheer->setDriver()->useRedisDriver();
```

```php
Cacheer::setDriver()->useRedisDriver();
```

Definir driver baseado em Array (memória):
```php

<?php

require_once __DIR__ . "/../vendor/autoload.php"; 

$Cacheer = new Cacheer();
$Cacheer->setDriver()->useArrayDriver();
```

```php
Cacheer::setDriver()->useArrayDriver();
```

