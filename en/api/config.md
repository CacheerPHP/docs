# Configuration

v6 has no ambient configuration — no `.env`, no global timezone, no implicit
schema. A `Cacheer` is exactly what you construct. Two things get configured: **how
the store is built** (named constructors) and **how values are stored**
(`PipelineConfig`).

## Named constructors

The common path needs one line:

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::inMemory();                    // ArrayStore
$cache = Cacheer::file('/var/cache/app');        // FileStore
$cache = Cacheer::database($pdo, 'cacheer');     // DatabaseStore (inject your PDO)
$cache = Cacheer::redis($connection);            // RedisStore (inject a connection)
```

Each persistent constructor accepts an optional `PipelineConfig` and `Clock`:

```php
$cache = Cacheer::file('/var/cache/app', $pipeline, $clock);
```

See [Cacheer & ScopedCacheer](./cache-functions.md#named-constructors) for the full
signatures, including the `tiered`, `resilient`, and `instrumented` decorators.

## `Cacheer::build()` — the fluent builder

For a richer setup, `Cacheer::build()` returns a `CacheerBuilder` that assembles a
store, a storage pipeline, and an optional default policy in one chain, then hands
back a ready cache. It's sugar over the named constructors, `PipelineConfig`, and
`CachePolicy` — nothing you can't do by hand, just shorter:

```php
$cache = Cacheer::build()
    ->file('/var/cache/app')                              // or ->inMemory() / ->database($pdo) / ->redis($conn)
    ->gzip()                                              // pipeline
    ->encryptWithPassphrases(['current' => $secret], 'current')
    ->maxValueBytes(2_000_000)
    ->defaultTtl('10 minutes')                            // policy
    ->jitter(0.10)
    ->serveStaleOnError('2 minutes')
    ->create();
```

| Group | Methods |
|---|---|
| Store | `inMemory()`, `file($dir)`, `database($pdo, $table = 'cacheer_store')`, `redis($connection, $prefix = 'cacheer')` |
| Pipeline | `json()`, `serializer()`, `gzip($level = 6)`, `compressor()`, `encrypt($keyring)`, `encryptWithPassphrases($map, $activeId)`, `encrypter()`, `maxValueBytes($n)` |
| Policy | `defaultTtl($ttl)`, `jitter($fraction)`, `negativeTtl($ttl)`, `serveStaleOnError($grace)` |
| Other | `clock($clock)` |

`create()` returns a plain `Cacheer`, or a `PolicyCacheer` when any policy method
was used (both expose the same read/write surface).

## `PipelineConfig` — the storage pipeline

`Silviooosilva\CacheerPhp\Config\PipelineConfig` is an immutable description of
how a value becomes bytes: **serialize → (compress) → (encrypt)**. Every `with*()`
returns a new instance, so a base config can be shared and specialized.

```php
use Silviooosilva\CacheerPhp\Config\PipelineConfig;
use Silviooosilva\CacheerPhp\Storage\Encryption\Keyring;
use Silviooosilva\CacheerPhp\Storage\Compat\V5PayloadReader;

$pipeline = PipelineConfig::default()          // PHP serializer, no compression/encryption
    ->withJsonSerializer()                     // or a custom Serializer
    ->withGzip(level: 6)                        // optional compression
    ->withKeyring(Keyring::fromPassphrases(['current' => $secret], 'current')) // AES-256-GCM
    ->withMaxValueBytes(2_000_000)              // reject oversized values on write
    ->withV5Reader(new V5PayloadReader());      // read v5 payloads during migration

$cache = Cacheer::file('/var/cache/app', $pipeline);
```

| Method | Effect |
|---|---|
| `default()` | PHP serialization, no compression, no encryption |
| `withSerializer(Serializer)` / `withJsonSerializer()` | Choose the serializer |
| `withCompressor(Compressor)` / `withGzip(int $level = 6)` | Add a compression stage |
| `withEncrypter(Encrypter)` / `withKeyring(Keyring)` | Add authenticated encryption |
| `withMaxValueBytes(int)` | Enforce a maximum serialized size on write |
| `withV5Reader(V5PayloadReader)` | Enable reading legacy v5 payloads |
| `codec()` | Build the ready `EnvelopeCodec` (stores call this) |

The pipeline is covered in depth in
[Compression & encryption](./compression-encryption.md) and the
[Encryption & compression guide](../guides/encryption-and-compression.md).

## Injecting your own dependencies

For full control, construct `Cacheer` directly:

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Support\SystemClock;

$cache = new Cacheer(
    store:    $store,
    clock:    new SystemClock(),
    executor: $deferredExecutor,   // for after-response stale refresh
    events:   $eventDispatcher,    // observability
);
```

Environment parsing belongs to your application or an optional bridge — not to the
core library.
