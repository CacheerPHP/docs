# Configuration

CacheerPHP 6 has **no ambient configuration**. It never loads `.env`, never
changes the global timezone, and never creates a database schema because it was
autoloaded. A `Cacheer` is exactly, and only, what you construct.

## The common path

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::inMemory();                 // ArrayStore — dependency-free
$cache = Cacheer::file('/var/cache/app');     // FileStore — persistent, dependency-free
$cache = Cacheer::database($pdo, 'cacheer');  // DatabaseStore — you own the PDO
$cache = Cacheer::redis($connection);         // RedisStore — you own the connection
```

That is all most applications need. Everything below is opt-in.

## Storing values safely: `PipelineConfig`

Persistent stores encode values through a pipeline you describe with an immutable
[`PipelineConfig`](../api/config.md):

```php
use Silviooosilva\CacheerPhp\Config\PipelineConfig;
use Silviooosilva\CacheerPhp\Storage\Encryption\Keyring;

$pipeline = PipelineConfig::default()
    ->withGzip()
    ->withKeyring(Keyring::fromPassphrases(['current' => $secret], 'current'))
    ->withMaxValueBytes(2_000_000);

$cache = Cacheer::file('/var/cache/app', $pipeline);
```

See [Encryption & compression](./encryption-and-compression.md) for details.

## Injecting a clock

Every constructor accepts a `Clock`. Production uses `SystemClock`; tests inject a
fake clock so expiry and stale windows are deterministic without `sleep()`.

```php
use Silviooosilva\CacheerPhp\Support\SystemClock;

$cache = Cacheer::file('/var/cache/app', clock: new SystemClock());
```

## Full dependency injection

When you need to control everything, construct `Cacheer` directly:

```php
$cache = new Cacheer(
    store:    $store,        // any Store
    clock:    $clock,        // Clock
    executor: $executor,     // DeferredExecutor — after-response stale refresh
    events:   $dispatcher,   // EventDispatcher — observability
);
```

## Where does environment configuration go?

In your application. Read env vars, build a PDO or Redis connection, and pass them
in. The library deliberately does not do this for you, so nothing happens as a
side effect of autoloading. A tiny bootstrap is enough:

```php
// bootstrap/cache.php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Stores\Support\PredisConnection;

return Cacheer::redis(new PredisConnection(new Predis\Client([
    'host' => $_ENV['REDIS_HOST'] ?? '127.0.0.1',
    'port' => (int) ($_ENV['REDIS_PORT'] ?? 6379),
])), prefix: $_ENV['CACHE_PREFIX'] ?? 'app');
```

The [operations CLI](./cli.md) uses the same idea: a `cacheer.config.php` that
returns a store.
