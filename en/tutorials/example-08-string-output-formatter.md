# Encryption & compression

Persistent stores encode values through a pipeline you describe with a
`PipelineConfig`. Here's a store that compresses and encrypts every value.

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Config\PipelineConfig;
use Silviooosilva\CacheerPhp\Storage\Encryption\Keyring;

$pipeline = PipelineConfig::default()
    ->withGzip()                                                     // compress large payloads
    ->withKeyring(Keyring::fromPassphrases(['current' => $secret], 'current')); // AES-256-GCM

$cache = Cacheer::file(__DIR__ . '/cache', $pipeline);

$cache->set('token', 'sensitive-data');   // stored compressed + encrypted
echo $cache->get('token');                // 'sensitive-data' — decrypted transparently
```

- Encryption is **authenticated**: tampered or truncated data is rejected on read,
  never returned. Requires `ext-openssl`.
- Compression is **bounded**, so a crafted blob can't exhaust memory. Requires
  `ext-zlib`.
- Rotate keys by adding a new id and marking it active — old entries still decrypt.

See the [Encryption & compression guide](../guides/encryption-and-compression.md).
