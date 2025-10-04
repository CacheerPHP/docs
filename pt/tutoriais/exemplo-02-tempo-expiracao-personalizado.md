# Exemplo 02 — Tempo de Expiração Personalizado (TTL)

```php

<?php
require_once __DIR__ . "/../vendor/autoload.php";

use Silviooosilva\\CacheerPhp\\Cacheer;

$options = [
    "cacheDir" =>  __DIR__ . "/cache",
    "expirationTime" => "2 hours"
];

$Cacheer = new Cacheer($options);

$cacheKey = 'daily_stats';
$dailyStats = [
    'visits' => 1500,
    'signups' => 35,
    'revenue' => 500.75,
];

Cacheer::putCache($cacheKey, $dailyStats, 'namespace', '2 hours');
$Cacheer->putCache($cacheKey, $dailyStats);

$cachedStats = $Cacheer->getCache($cacheKey, 'namespace', '2 hours');
```

Formatos aceitos: seconds, minutes, hours.

