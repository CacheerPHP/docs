# Políticas

Uma `CachePolicy` agrupa comportamento transversal de escrita — um TTL padrão, jitter
de TTL, cache negativo e serve-stale-on-error — e o aplica de forma consistente a toda
escrita e `remember()` de um cache. Leituras passam direto.

```php
use Silviooosilva\CacheerPhp\Config\CachePolicy;

$cache = $cache->withPolicy(
    CachePolicy::defaults()
        ->withTtl('10 minutes')
        ->withJitter(0.10)
        ->withNegativeTtl('30 seconds')
        ->withServeStaleOnError('2 minutes'),
);
```

`withPolicy()` retorna um `PolicyCache` que embrulha o original — a política é
imutável, e cada `with*()` retorna uma nova instância.

## TTL padrão — `withTtl()`

Define o TTL usado quando uma escrita não especifica um. Um TTL por chamada ainda
vence.

```php
$policy = CachePolicy::defaults()->withTtl('10 minutes');
$cache->set('k', $v);              // guardado por 10 minutos
$cache->set('k', $v, ttl: 3600);   // TTL explícito sobrescreve a política
```

## Jitter de TTL — `withJitter()`

Espalha as expirações para que um lote de entradas escritas juntas não expire tudo no
mesmo instante (o que causaria um estampede sincronizado). O jitter é uma fração do
TTL, aplicada por um randomizador injetável para os testes ficarem determinísticos.

```php
$policy = CachePolicy::defaults()->withTtl('10 minutes')->withJitter(0.10);
// cada escrita cai em ~9–10 minutos
```

## Cache negativo — `withNegativeTtl()`

Cacheia resultados "vazios" (ex.: `null`, `[]`) por um tempo **menor** que o normal,
para que um registro ausente seja re-tentado mais cedo que um populado — protegendo seu
banco de buscas repetidas por coisas que não existem, sem prender a ausência por muito
tempo.

```php
$policy = CachePolicy::defaults()->withTtl('1 hour')->withNegativeTtl('30 seconds');
$cache->remember('user:999', null, fn () => $users->find(999)); // null cacheado 30s, um hit 1h
```

## Serve-stale-on-error — `withServeStaleOnError()`

Define uma janela de graça após a expiração lógica durante a qual, **se um refresh
falhar**, o último valor bom é servido em vez de propagar o erro. Isso é sobre falhas,
complementando o [`flexible()`](./stale-while-revalidate.md), que é sobre latência.

```php
$policy = CachePolicy::defaults()->withTtl('5 minutes')->withServeStaleOnError('2 minutes');
```

## Determinismo

O comportamento da política é totalmente determinístico sob um clock falso e um
randomizador injetado, então jitter, cache negativo e janelas de graça são testáveis
ao segundo.
