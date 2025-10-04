# Exemplo 07 — Saída Array

```php

<?php
require_once  __DIR__  .  "/../vendor/autoload.php";
use Silviooosilva\\CacheerPhp\\Cacheer;

$options = [
  "cacheDir" => __DIR__  .  "/cache",
];

$Cacheer = new Cacheer($options, $formatted = true);

$cacheKey = 'user_profile_1234';
$userProfile = ['id' => 123, 'name' => 'John Doe'];

Cacheer::putCache($cacheKey, $userProfile);
$Cacheer->putCache($cacheKey, $userProfile);

$cachedProfile = $Cacheer->getCache($cacheKey, $namespace, $ttl)->toArray();
```

