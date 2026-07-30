# Configuração

O CacheerPHP 6 **não tem configuração ambiente**. Nunca carrega `.env`, nunca muda o
timezone global e nunca cria um schema de banco por ter sido autoloadeado. Um
`Cacheer` é exatamente, e apenas, o que você constrói.

## O caminho comum

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::inMemory();                 // ArrayStore — sem dependências
$cache = Cacheer::file('/var/cache/app');     // FileStore — persistente, sem dependências
$cache = Cacheer::database($pdo, 'cacheer');  // DatabaseStore — você é dono do PDO
$cache = Cacheer::redis($connection);         // RedisStore — você é dono da conexão
```

É tudo que a maioria dos apps precisa. O resto abaixo é opcional.

## Armazenando valores com segurança: `PipelineConfig`

Stores persistentes codificam valores por um pipeline descrito com um
[`PipelineConfig`](../api/configuracao.md) imutável:

```php
use Silviooosilva\CacheerPhp\Config\PipelineConfig;
use Silviooosilva\CacheerPhp\Storage\Encryption\Keyring;

$pipeline = PipelineConfig::default()
    ->withGzip()
    ->withKeyring(Keyring::fromPassphrases(['current' => $secret], 'current'))
    ->withMaxValueBytes(2_000_000);

$cache = Cacheer::file('/var/cache/app', $pipeline);
```

Veja [Criptografia e compressão](./criptografia-e-compressao.md) para detalhes.

## Injetando um clock

Todo construtor aceita um `Clock`. Produção usa `SystemClock`; testes injetam um
clock falso para que expiração e janelas de stale sejam determinísticas sem
`sleep()`.

```php
use Silviooosilva\CacheerPhp\Support\SystemClock;

$cache = Cacheer::file('/var/cache/app', clock: new SystemClock());
```

## Injeção de dependências completa

Quando precisar controlar tudo, construa `Cacheer` diretamente:

```php
$cache = new Cacheer(
    store:    $store,        // qualquer Store
    clock:    $clock,        // Clock
    executor: $executor,     // DeferredExecutor — stale refresh após a resposta
    events:   $dispatcher,   // EventDispatcher — observabilidade
);
```

## Onde vai a configuração de ambiente?

Na sua aplicação. Leia as variáveis de ambiente, construa um PDO ou conexão Redis, e
passe-os. A biblioteca deliberadamente não faz isso por você, para que nada aconteça
como efeito colateral do autoload. Um bootstrap pequeno basta:

```php
// bootstrap/cache.php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Stores\Support\PredisConnection;

return Cacheer::redis(new PredisConnection(new Predis\Client([
    'host' => $_ENV['REDIS_HOST'] ?? '127.0.0.1',
    'port' => (int) ($_ENV['REDIS_PORT'] ?? 6379),
])), prefix: $_ENV['CACHE_PREFIX'] ?? 'app');
```

A [CLI de operações](./cli.md) usa a mesma ideia: um `cacheer.config.php` que retorna
uma store.
