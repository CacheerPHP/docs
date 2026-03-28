## API Reference — TimeBuilder

TimeBuilder provides a fluid and chainable way of defining time periods in a more intuitive way and without typing errors. 

It allows expirationTime and flushAfter values to be passed directly as integers or defined using chained methods such as day(1), week(2), etc.

TimeBuilder can be chained from File, Redis and Database builders to define values for `expirationTime()` and `flushAfter()`.

#### Simple use (File)

```php

<?php
OptionBuilder::forFile()
    ->expirationTime('1 day')
    ->build();
```
Or use TimeBuilder's chained approach:

```php

<?php
OptionBuilder::forFile()
    ->expirationTime()->day(1)
    ->build();
```

#### Use with Redis and Database

```php

<?php
// Redis default TTL of 10 minutes
OptionBuilder::forRedis()
  ->expirationTime()->minute(10)
  ->build();

// Database flush interval of 1 week
OptionBuilder::forDatabase()
  ->flushAfter()->week(1)
  ->build();
```

#### Available methods

Each method allows you to set a specific time interval.

| Method        | Description                      | Example       |
|--------------|--------------------------------|--------------|
| `second($value)` | Set time in seconds  | `->second(30)` |
| `minute($value)` | Set time in minutes  | `->minute(15)` |
| `hour($value)`   | Set time in hours    | `->hour(3)`    |
| `day($value)`    | Set time in days     | `->day(7)`     |
| `week($value)`   | Set time in weeks    | `->week(2)`    |
| `month($value)`  | Set time in months   | `->month(1)`   |
| `year($value)`   | Set time in years    | `->year(1)`    |

#### Full Example

```php

<?php
$Options = OptionBuilder::forFile()
    ->dir(__DIR__ . '/cache')
    ->expirationTime()->week(1)
    ->flushAfter()->minute(30)
    ->build();

var_dump($Options);
```

**Expected Output**

```php
[
    "cacheDir" => "/path/to/cache",
    "expirationTime" => "1 week",
    "flushAfter" => "30 minutes"
]
```

Now you can set expiration and flush times without having to remember exact strings. 🚀
