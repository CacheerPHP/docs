# Example 03 — Cleaning and Flushing

```php

<?php
require_once __DIR__ . "/../vendor/autoload.php";

use Silviooosilva\\CacheerPhp\\Cacheer;

$options = [
    "cacheDir" =>  __DIR__ . "/cache",
];

$Cacheer = new Cacheer($options);

Cacheer::flushCache();

$cacheKey = 'user_profile_123';
if ($Cacheer->clearCache($cacheKey)) {
    echo $Cacheer->getMessage();
}

if ($Cacheer->flushCache()) {
    echo $Cacheer->getMessage();
}
```

