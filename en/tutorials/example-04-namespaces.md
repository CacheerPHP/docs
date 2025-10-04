# Example 04 — Namespaces

```php

<?php
require_once __DIR__ . "/../vendor/autoload.php";

use Silviooosilva\\CacheerPhp\\Cacheer;

$options = [
    "cacheDir" =>  __DIR__ . "/cache",
];

$Cacheer = new Cacheer($options);

$namespace = 'session_data_01';
$cacheKey = 'session_456';
$sessionData = [
    'user_id' => 456,
    'login_time' => time(),
];

Cacheer::putCache($cacheKey, $sessionData, $namespace);
$Cacheer->putCache($cacheKey, $sessionData, $namespace);

$cachedSessionData = $Cacheer->getCache($cacheKey, $namespace);
```

