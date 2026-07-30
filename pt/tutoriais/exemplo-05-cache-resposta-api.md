# Cacheando uma resposta de API

`remember()` retorna o valor cacheado, ou o calcula uma vez num miss e o guarda.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

$weather = $cache->remember('weather:lisbon', ttl: '15 minutes', callback: function () {
    $json = file_get_contents('https://api.example.com/weather?city=lisbon');
    return json_decode($json, true);
});
```

- Num hit, a chamada HTTP nunca acontece.
- Num miss concorrente, o callback roda **uma vez** entre os workers (single-flight),
  não uma por requisição — sem estampede. Veja
  [Proteção contra estampede](./exemplo-18-protecao-stampede-swr.md).
- Cacheie para sempre passando `ttl: null`.

Para endpoints quentes onde você nunca quer que um usuário espere por um refresh, use
[`flexible()`](./exemplo-18-protecao-stampede-swr.md).
