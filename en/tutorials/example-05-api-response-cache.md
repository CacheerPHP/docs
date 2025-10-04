# Example 05 — API Response Cache

```php

<?php
require_once __DIR__ . "/../vendor/autoload.php";

use Silviooosilva\\CacheerPhp\\Cacheer;

$options = [
    "cacheDir" =>  __DIR__ . "/cache",
];

$Cacheer = new Cacheer($options);

$apiUrl = 'https://jsonplaceholder.typicode.com/posts';
$cacheKey = 'api_response_' . md5($apiUrl);

$cachedResponse = $Cacheer->getCache($cacheKey);

if ($Cacheer->has($cacheKey)) {
    $response = $cachedResponse;
} else {
    $response = file_get_contents($apiUrl);
    $Cacheer->putCache($cacheKey, $response);
}

$data = json_decode($response, true);
```

