# Store resiliente (fallback + circuit breaker)

Um cache resiliente continua funcionando quando sua store primária tem um dia ruim.
Ele serve de uma store **primária** e, quando a primária começa a falhar, aciona um
**circuit breaker** e serve de um **fallback** — sem martelar o backend quebrado.

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Stores\ArrayStore;

$cache = Cacheer::resilient(
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

$cache = Cacheer::resilient($primary, $fallback, breaker: new CircuitBreaker(/* limiares */));
```

## Falha fechada, nunca errada

Quando o breaker está aberto e o fallback também dá miss, o resultado é um **miss** —
nunca dado stale ou fabricado. Resiliência compra disponibilidade, não uma
flexibilização da correção.

## O que vai para o fallback, e o que não vai

- **Só quedas vão para o fallback.** Uma falha de PDO, Redis ou I/O conta para o
  breaker e cai no fallback. Um bug (`TypeError`, um argumento inválido) ou dado ruim
  (payload corrompido ou grande demais, overflow de contador) é relançado como está —
  ir para o fallback só o esconderia.
- **O resultado da primária é o que vale.** Escritas vão para a primária e são
  espelhadas no fallback em regime de melhor esforço (uma queda do fallback não faz a
  escrita falhar). Só enquanto a primária está indisponível o resultado do fallback
  prevalece.
- **Contadores e locks nunca vão para o fallback.** `increment()`, `compareAndSwap()`
  e `lock()` rodam só na primária, porque um segundo contador ou lock independente no
  fallback a contradiria. Com a primária fora, eles falham fechados: contadores lançam
  `StoreOperationFailedException`, e locks informam "não adquirido" na hora, então o
  `remember()` simplesmente calcula sem single-flight.
- **A recuperação não ressuscita dados antigos.** Chaves gravadas ou removidas só no
  fallback durante a queda são invalidadas na primária antes de ela voltar a servir,
  e um `clearScope()`/`clearTag()` feito na queda é repetido. Depois de um `clear()`
  na queda, ou de mais de 1.000 dessas escritas, a primária é limpa. Isso é
  registrado por processo, então cada worker reconcilia as próprias escritas.

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
uma queda do L2 degrade para o fallback em vez de dar erro:

```php
use Silviooosilva\CacheerPhp\Stores\ResilientStore;

$shared = new ResilientStore($redisStore, new ArrayStore($clock), clock: $clock);
$cache  = Cacheer::tiered(new ArrayStore($clock), $shared, clock: $clock);
```

Veja [Observabilidade](./observabilidade.md) para emitir eventos `cache.failure`
quando a primária cai.
