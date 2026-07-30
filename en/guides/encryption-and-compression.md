# Encryption & compression

Persistent stores encode every value through a pipeline —
**serialize → compress → encrypt** — and store the result in a versioned,
authenticated envelope. You describe the pipeline with an immutable
[`PipelineConfig`](../api/config.md) and pass it to the store.

```php
use Silviooosilva\CacheerPhp\Config\PipelineConfig;
use Silviooosilva\CacheerPhp\Storage\Encryption\Keyring;
use Silviooosilva\CacheerPhp\Cacheer;

$pipeline = PipelineConfig::default()
    ->withGzip()
    ->withKeyring(Keyring::fromPassphrases(['current' => $secret], 'current'));

$cache = Cacheer::file('/var/cache/app', $pipeline);
```

## Compression

Gzip is optional and most useful for large payloads (big arrays, HTML fragments).
Decompression is **bounded** by `withMaxValueBytes()`, so a crafted blob cannot
force unbounded memory use.

```php
$pipeline = PipelineConfig::default()->withGzip(level: 6)->withMaxValueBytes(2_000_000);
```

## Encryption — authenticated AES-256-GCM

Encryption is **authenticated**: on read, tampered or truncated ciphertext is
rejected with a typed exception — it is never returned as data. Requires
`ext-openssl`.

```php
$keyring = Keyring::fromPassphrases(['2026' => $secret], activeId: '2026');
$pipeline = PipelineConfig::default()->withKeyring($keyring);
```

### Key rotation

The envelope records which key id encrypted each value. To rotate, add a new key,
make it active, and keep the old id available for reads:

```php
$keyring = Keyring::fromPassphrases(
    ['2025' => $oldSecret, '2026' => $newSecret],
    activeId: '2026',   // new writes use 2026
);
```

Old entries written under `2025` keep decrypting; new writes use `2026`. No flush,
no downtime. Retire the old id once nothing old remains.

## Order matters

The pipeline compresses **before** it encrypts (encrypted data doesn't compress).
CacheerPHP handles the ordering for you — you just declare the stages.

## Combining with a max size

```php
$pipeline = PipelineConfig::default()
    ->withGzip()
    ->withKeyring($keyring)
    ->withMaxValueBytes(1_000_000); // ValueTooLargeException on oversized writes
```

## Guarantees and limits

- Tampering, a wrong key, truncation, or an over-limit value produce
  **deterministic, typed failures** — never silent corruption or unauthenticated
  data.
- The default pipeline does **not** encrypt or compress. Opt in explicitly; never
  cache secrets without encryption.
- See [KNOWN_LIMITATIONS](../updating/index.md) and the
  [Compression & encryption reference](../api/compression-encryption.md) for the
  envelope format and v5 read compatibility.
