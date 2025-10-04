# Example 09 — Auto Flush with `flushAfter`

```php

<?php
require_once  __DIR__  .  "/../vendor/autoload.php";
use Silviooosilva\\CacheerPhp\\Cacheer;

$options = [
  "cacheDir" => __DIR__  .  "/cache",
  "flushAfter" => "1 week"
];

$Cacheer = new Cacheer($options);

$cacheKey = 'user_profile_1234';
$userProfile = ['id' => 123, 'name' => 'John Doe'];

Cacheer::putCache($cacheKey, $userProfile);
$Cacheer->putCache($cacheKey, $userProfile);

$cachedProfile = $Cacheer->getCache($cacheKey);
```

Accepted intervals for `flushAfter`: seconds, minutes, hours, days, weeks, months, years.

