# Configuration

v6 has no ambient configuration — no `.env`, no global timezone, no implicit
schema. A `Cache` is exactly what you construct. Two things get configured: **how
the store is built** (named constructors) and **how values are stored**
(`PipelineConfig`).

## Named constructors

The common path needs one line:

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;

$cache = Cache::inMemory();                    // ArrayStore
$cache = Cache::file('/var/cache/app');        // FileStore
$cache = Cache::database($pdo, 'cacheer');     // DatabaseStore (inject your PDO)
$cache = Cache::redis($connection);            // RedisStore (inject a connection)
```

Each persistent constructor accepts an optional `PipelineConfig` and `Clock`:

```php
$cache = Cache::file('/var/cache/app', $pipeline, $clock);
```

See [Cache & ScopedCache](./cache-functions.md#named-constructors) for the full
signatures, including the `tiered`, `resilient`, and `instrumented` decorators.

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

$cache = Cache::file('/var/cache/app', $pipeline);
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

For full control, construct `Cache` directly:

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Support\SystemClock;

$cache = new Cache(
    store:    $store,
    clock:    new SystemClock(),
    executor: $deferredExecutor,   // for after-response stale refresh
    events:   $eventDispatcher,    // observability
);
```

Environment parsing belongs to your application or an optional bridge — not to the
core library.
