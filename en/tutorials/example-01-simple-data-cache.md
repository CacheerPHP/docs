# Example 01 — Simple Data Cache

```php

<?php
require_once __DIR__ . "/../vendor/autoload.php";

use Silviooosilva\\CacheerPhp\\Cacheer;

$options = [
    "cacheDir" =>  __DIR__ . "/cache",
];

$Cacheer = new Cacheer($options);

$cacheKey = 'user_profile_1234';
$userProfile = [
    'id' => 123,
    'name' => 'John Doe',
    'email' => 'john.doe@example.com',
];

Cacheer::putCache($cacheKey, $userProfile);
$Cacheer->putCache($cacheKey, $userProfile);

$cachedProfile = $Cacheer->getCache($cacheKey);

if ($Cacheer->has($cacheKey)) {
    var_dump($cachedProfile);
}
```

