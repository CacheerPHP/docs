# Exemplo 13 — Cache de Valores Falsy

*Novo na v5.0.0*

O CacheerPHP v5.0.0 guarda e devolve corretamente valores **falsy** como `0`, `''`, `false` e `[]`. Em versões anteriores, estes eram tratados erradamente como cache misses.

## O Problema (v4.x)

```php
$cache->putCache('counter', 0);
$value = $cache->getCache('counter');
// v4: $value era tratado como "não encontrado" por causa de verificações !empty()
```

## A Correção (v5.0.0)

A v5 usa `isSuccess()` internamente em vez de `!empty()`, pelo que qualquer valor PHP — incluindo os falsy — é devolvido corretamente.

## Guardar e Obter Valores Falsy

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useArrayDriver();

// Inteiro zero
$cache->putCache('zero', 0);
echo $cache->getCache('zero'); // 0

// String vazia
$cache->putCache('empty_str', '');
var_dump($cache->getCache('empty_str')); // string(0) ""

// Booleano false
$cache->putCache('flag', false);
var_dump($cache->getCache('flag')); // bool(false)

// Array vazio
$cache->putCache('list', []);
var_dump($cache->getCache('list')); // array(0) {}
```

## remember() com Valores Falsy

O método `remember()` já não volta a executar o callback quando o valor em cache é falsy:

```php
$callCount = 0;

$value = $cache->remember('counter', 3600, function () use (&$callCount) {
    $callCount++;
    return 0; // falsy mas válido
});

// Segunda chamada — o callback NÃO é invocado outra vez
$value = $cache->remember('counter', 3600, function () use (&$callCount) {
    $callCount++;
    return 0;
});

echo $callCount; // 1 (e não 2)
echo $value;     // 0
```

## increment() e decrement() a Partir de Zero

```php
$cache->putCache('hits', 0);

$cache->increment('hits');
echo $cache->getCache('hits'); // 1

$cache->putCache('balance', 0);
$cache->decrement('balance');
echo $cache->getCache('balance'); // -1
```
