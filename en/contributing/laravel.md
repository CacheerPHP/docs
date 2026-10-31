# Contributing to CacheerPHP for Laravel

Open an issue or pull request in
[CacheerPHP/laravel](https://github.com/CacheerPHP/laravel).
The driver supports PHP 8.3+ and Laravel 12 or 13.

## Setup and checks

```sh
git clone https://github.com/CacheerPHP/laravel.git
cd laravel
composer install
composer test
```

`composer test` validates Composer metadata and runs Rector in dry-run mode,
Pint, PHPStan, and PHPUnit. CI also checks lower dependency bounds and installs
the distribution ZIP into clean Laravel 12 and 13 applications without
development dependencies, exercising discovery, cached configuration, and
published migrations.

## Redis integration tests

Point the tests at a disposable Redis server. They use database **15** and a
separate key prefix:

```sh
CACHEER_TEST_REDIS_PORT=6379 vendor/bin/phpunit
CACHEER_TEST_REDIS_CLIENT=phpredis CACHEER_TEST_REDIS_PORT=6379 vendor/bin/phpunit --filter redis
```

Predis is the default test client. The second command requires `ext-redis`.
Set `CACHEER_TEST_REDIS_HOST` if the server is not on `127.0.0.1`.

See [Laravel features](../laravel/framework-features.md) and
[differences and limitations](../laravel/limitations.md) for the driver contract.
