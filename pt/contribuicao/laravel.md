# Contribuir para o CacheerPHP para Laravel

Abra uma issue ou um pull request em
[CacheerPHP/cacheerphp-laravel](https://github.com/CacheerPHP/cacheerphp-laravel).
O driver suporta PHP 8.3+ e Laravel 12 ou 13.

## Preparação e verificações

```sh
git clone https://github.com/CacheerPHP/cacheerphp-laravel.git
cd cacheerphp-laravel
composer install
composer test
```

`composer test` valida os metadados do Composer e executa o Rector sem aplicar
alterações, o Pint, o PHPStan e o PHPUnit. O CI também verifica os limites
inferiores das dependências e instala o ZIP de distribuição em aplicações
Laravel 12 e 13 limpas, sem dependências de desenvolvimento, para testar a
descoberta do pacote, a configuração em cache e as migrations publicadas.

## Testes de integração com Redis

Aponte os testes para um servidor Redis descartável. Usam a base de dados
**15** e um prefixo de chaves separado:

```sh
CACHEER_TEST_REDIS_PORT=6379 vendor/bin/phpunit
CACHEER_TEST_REDIS_CLIENT=phpredis CACHEER_TEST_REDIS_PORT=6379 vendor/bin/phpunit --filter redis
```

O cliente predefinido dos testes é o Predis. O segundo comando requer
`ext-redis`. Defina `CACHEER_TEST_REDIS_HOST` se o servidor não estiver em
`127.0.0.1`.

Consulte as [funcionalidades do Laravel](../laravel/framework-features.md) e as
[diferenças e limitações](../laravel/limitations.md) para o contrato do driver.
