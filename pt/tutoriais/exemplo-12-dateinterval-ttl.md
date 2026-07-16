# Exemplo 12 — DateInterval e TTL null

*Novo na v5.0.0*

Todos os parâmetros de TTL aceitam agora quatro formatos: `int` (segundos), `string` (legível), `\DateInterval` e `null` (para sempre).

## TTL com DateInterval

Use o `\DateInterval` nativo do PHP para TTLs tipados e amigáveis para o IDE:

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useArrayDriver();

// 45 minutos
$cache->putCache('session', $data, ttl: new DateInterval('PT45M'));

// 1 dia
$cache->putCache('report', $data, ttl: new DateInterval('P1D'));

// 2 horas e 30 minutos
$cache->putCache('token', $data, ttl: new DateInterval('PT2H30M'));
```

## TTL null — Guardar Para Sempre

Passar `null` guarda o item sem expiração (`PHP_INT_MAX` segundos):

```php
$cache->putCache('app_version', 'v5.0.0', ttl: null);
```

## Funciona em Todo o Lado

`\DateInterval` e `null` são aceites por todos os métodos que recebem TTL:

```php
// putCache
$cache->putCache('key', 'data', ttl: new DateInterval('PT30M'));

// add (escrita condicional)
$cache->add('lock', getmypid(), ttl: new DateInterval('PT1M'));

// remember (obter-ou-calcular)
$cache->remember('stats', new DateInterval('PT5M'), fn() => computeStats());

// renewCache (renovar TTL)
$cache->renewCache('key', new DateInterval('PT2H'));
```

## Comparação de Todos os Formatos

```php
// Inteiro — segundos
$cache->putCache('k', 'v', ttl: 3600);

// String — legível
$cache->putCache('k', 'v', ttl: '2 hours');

// DateInterval — nativo do PHP
$cache->putCache('k', 'v', ttl: new DateInterval('PT2H'));

// null — para sempre
$cache->putCache('k', 'v', ttl: null);
```
