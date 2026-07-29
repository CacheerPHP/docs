# Store resiliente (fallback + circuit breaker)

Um cache resiliente continua funcionando quando sua store primária tem um dia ruim.
Ele serve de uma store **primária** e, quando a primária começa a falhar, aciona um
**circuit breaker** e serve de um **fallback** — sem martelar o backend quebrado.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Stores\ArrayStore;

$cache = Cache::resilient(
    primary:  $redisStore,            // normalmente serve tudo
    fallback: new ArrayStore($clock), // assume quando o Redis está ruim
);
```

## O circuit breaker

O breaker tem três estados:

- **Fechado** — normal. Requisições vão à primária. Falhas são contadas.
- **Aberto** — após falhas demais, o breaker abre: requisições pulam a primária e vão
  direto ao fallback, para um backend lento ou fora do ar não arrastar cada
  requisição a um timeout.
- **Meio-aberto** — após um resfriamento, uma requisição de sondagem passa. Sucesso
  fecha o breaker; falha reabre.

Ajuste passando seu próprio `CircuitBreaker`:

```php
use Silviooosilva\CacheerPhp\Support\CircuitBreaker;

$cache = Cache::resilient($primary, $fallback, breaker: new CircuitBreaker(/* limiares */));
```

## Falha fechada, nunca errada

Quando o breaker está aberto e o fallback também dá miss, o resultado é um **miss** —
nunca dado stale ou fabricado. Resiliência compra disponibilidade, não uma
flexibilização da correção.

## Saúde degradada, sem segredos

Você pode observar o estado do breaker (fechado/aberto/meio-aberto) para alimentar um
health check ou dashboard. A saúde exposta é apenas o estado do breaker — nunca
vazando strings de conexão ou credenciais.

## Resiliência vs. camadas

- [`TieredStore`](./cache-em-camadas.md) é sobre **velocidade**: um miss na primária
  (L2) é um miss real, e o L1 só torna hits rápidos.
- `ResilientStore` é sobre **falha**: uma *queda* da primária é mascarada pelo
  fallback.

Eles se compõem — um L1 local, um L2 compartilhado, e um wrapper resiliente para que
uma queda do L2 degrade para o fallback em vez de dar erro.

Veja [Observabilidade](./observabilidade.md) para emitir eventos `cache.failure`
quando a primária cai.
