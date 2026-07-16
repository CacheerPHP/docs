# Exemplo 17 — Bloqueios Distribuídos & Contadores Atómicos

*Novo na v5.2.0*

Duas funcionalidades relacionadas para acesso concorrente seguro:

- **`lock()`** — um mutex nomeado, suportado pelo driver, para que só um processo execute uma secção crítica de cada vez.
- **`increment()` / `decrement()` atómicos** — contadores que nunca perdem atualizações sob concorrência.

Ambos funcionam entre processos nos drivers File, Database e Redis.

## Lock Distribuído

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useFileDriver();

// Só um processo executa o callback de cada vez; os outros recebem `false`.
$result = $cache->lock('rebuild-report', ttl: 30)->get(function () {
    // trabalho caro que não pode correr duas vezes
    return rebuildReport();
});

if ($result === false) {
    echo "Outro processo já está a reconstruir o relatório.\n";
}
```

### Esperar pelo lock em vez de saltar

```php
// Espera até 5 segundos pelo lock e depois executa em exclusivo.
$charged = $cache->lock("invoice:{$id}", 30)->block(5, fn () => chargeInvoice($id));
```

### Acquire / release manual

```php
$lock = $cache->lock('nightly-job', 120);

if (! $lock->acquire()) {
    exit("O job já está a correr noutro lado.\n");
}

try {
    runNightlyJob();
} finally {
    $lock->release();
}
```

## Contadores Atómicos

`increment()` e `decrement()` leem o valor atual, alteram-no e voltam a escrevê-lo. Com pedidos concorrentes, esse ciclo ler-modificar-escrever entra em corrida e **perde atualizações**. Em qualquer driver com locking, o CacheerPHP serializa agora cada atualização de contador num lock por chave, pelo que cada incremento é aplicado exatamente uma vez.

```php
$cache->putCache('views:post:1', 0);

// Seguro mesmo quando muitos pedidos chegam a esta linha ao mesmo tempo.
$cache->increment('views:post:1');       // +1
$cache->increment('views:post:1', 10);   // +10
$cache->decrement('views:post:1', 3);    // -3
```

As assinaturas e o comportamento de criação em caso de miss não mudaram — só a garantia de concorrência é nova:

```php
// Chave em falta, sem default → false, nada é escrito
$cache->increment('absent');                       // false

// Chave em falta, com default → criada como (default + quantidade)
$cache->increment('hits', 1, '', 0);               // cria 'hits' = 1
$cache->increment('budget', 10, '', 100);          // em falta → 110
$cache->increment('rate', 1, '', 0, '1 hour');     // criação em miss com TTL de 1h
```

## Caso de Uso: Rate Limiting

```php
function tooManyAttempts(Cacheer $cache, string $key, int $max, int $window): bool
{
    // Contador atómico, criado no primeiro acesso com janela TTL deslizante.
    $cache->increment($key, 1, '', 0, $window);
    return (int) $cache->getCache($key) > $max;
}

if (tooManyAttempts($cache, "login:{$ip}", max: 5, window: 60)) {
    http_response_code(429);
    exit("Demasiadas tentativas. Tente novamente mais tarde.\n");
}
```

## Ver também

- [Bloqueios Distribuídos — referência da API](../api/locks.md)
- [Funções de Cache → increment() / decrement()](../api/funcoes-cache.md)
