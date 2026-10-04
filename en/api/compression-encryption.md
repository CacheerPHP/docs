# Compression & encryption — the storage envelope

Persistent stores (`FileStore`, `DatabaseStore`, `RedisStore`) run every value
through a pipeline and store the result in a **versioned, authenticated
envelope**. The pipeline is configured with a [`PipelineConfig`](./config.md).

## The envelope

```text
serialize  ->  compress (optional)  ->  encrypt (optional)  ->  Envelope
```

The `Envelope` (`Silviooosilva\CacheerPhp\Storage\Envelope`) records the format
version, serializer id, compressor id, encrypter id, and key id, then the payload.
It is NUL-prefixed with a magic marker so it can never be confused with a v5
payload. `EnvelopeCodec` encodes and decodes it; failures are deterministic and
typed — a tampered, truncated, over-limit, or unrecognized blob raises an
exception instead of returning corrupt or unauthenticated data.

## Serializers

- `PhpSerializer` (default) — native PHP serialization; handles any serializable
  value, including objects.
- `JsonSerializer` — portable JSON; throws on values JSON cannot represent.

```php
$pipeline = PipelineConfig::default()->withJsonSerializer();
```

`PhpSerializer` allows every class by default, so without encryption the backend
is trusted: anyone who can write to it can plant objects that are unserialized on
read. If the backend is shared, enable encryption or restrict the classes:

```php
use Silviooosilva\CacheerPhp\Storage\Serializer\PhpSerializer;

$pipeline = PipelineConfig::default()->withSerializer(new PhpSerializer(allowedClasses: false));
```

## Compression

Optional gzip, useful for large payloads. Decompression is **bounded** (it honors
`withMaxValueBytes`), so a malicious blob cannot force unbounded memory use.

```php
$pipeline = PipelineConfig::default()->withGzip(level: 6);
```

## Encryption — authenticated AES-256-GCM

Encryption is authenticated: decoding rejects tampered or truncated ciphertext
rather than returning it. Keys are managed by a `Keyring` that supports rotation.

```php
use Silviooosilva\CacheerPhp\Storage\Encryption\Keyring;

// Derive keys from passphrases (a key id maps to each), with one active id.
$keyring = Keyring::fromPassphrases(
    ['2025' => $oldSecret, '2026' => $newSecret],
    activeId: '2026',
);

$pipeline = PipelineConfig::default()->withKeyring($keyring);
```

- New writes use the **active** key; the key id is stored in the envelope.
- Old entries written under a retired key still decrypt as long as that key id
  remains in the keyring — so you can rotate without a flush.
- An encrypting pipeline reads **only** encrypted envelopes: a plaintext one
  throws `UnsupportedEnvelopeException`, so nobody can bypass authentication by
  writing an envelope that declares "no encryption". When you turn encryption on
  for an existing cache, clear it first (or use a new keyspace).
- Requires `ext-openssl`.

> Never cache secrets in a store without encryption enabled. The default pipeline
> does not encrypt.

## Size limits

```php
$pipeline = PipelineConfig::default()->withMaxValueBytes(2_000_000);
```

A value whose serialized form exceeds the limit throws `ValueTooLargeException`
on write. The same limit applies on read to every pipeline — plain, compressed, or
encrypted — before anything is unserialized, and decompression stops as soon as
it is exceeded. `0` means no limit; a negative limit throws
`InvalidArgumentException`.

## Non-envelope data

`decode()` accepts only v6 envelopes. Any other blob — including a payload written
by CacheerPHP v5 — throws `UnsupportedEnvelopeException` instead of being decoded,
whatever stages the pipeline has. v6 does not read v5's cached data; see the
[migration guide](../updating/index.md#5-cached-data-v6-starts-cold).
