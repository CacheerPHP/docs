# API Reference — Compression & Encryption

CacheerPHP provides built-in compression and encryption that can be enabled independently or combined. Both work transparently across all four drivers.

---

## Compression

### `useCompression()`

Enables or disables data compression before storage. When enabled, data is serialized and compressed using `gzcompress`, reducing on-disk/in-memory size.

```php
$cache->useCompression();        // enable
$cache->useCompression(false);   // disable
```

Compression is applied **before** encryption when both are active.

---

## Encryption

### `useEncryption()`

Activates AES-256-CBC encryption. Provide a secret key to encrypt and decrypt cached data.

```php
$cache->useEncryption('your-32-byte-secret-key');
```

### v5.0.0 Security Improvement — Random IV

In v4.x, the initialization vector (IV) was derived deterministically from the encryption key using `hash('sha256', $key)`. This meant identical payloads always produced identical ciphertexts — a cryptographic weakness (breaks CBC semantic security).

**v5.0.0 generates a fresh random 16-byte IV** for every `putCache()` call using `openssl_random_pseudo_bytes(16)`. The IV is prepended to the ciphertext and the entire blob is base64-encoded for storage. On read, the IV is extracted from the first 16 bytes and used for decryption.

This means:
- Same data stored twice produces **different** ciphertexts (good)
- No API change required — encryption/decryption is fully transparent
- **Breaking**: data encrypted with v4 cannot be decrypted by v5 (different format). Flush encrypted caches before upgrading.

### Combining Both

Compress first, then encrypt — for smaller and secure payloads:

```php
$cache->useCompression()->useEncryption('your-secret-key');
```

### Verifying Feature State

Use `stats()` to check which features are active:

```php
$info = $cache->stats();
echo $info['compression'];  // true or false
echo $info['encryption'];   // true or false
```

---

## Example

```php
<?php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer(['cacheDir' => __DIR__ . '/cache']);
$cache->useCompression();
$cache->useEncryption('my-32-byte-secret-key-for-aes!!');

$sensitiveData = [
    'credit_card' => '4111 1111 1111 1111',
    'cvv'         => '123',
];

// Data is compressed, then encrypted, then stored
$cache->putCache('payment', $sensitiveData);

// Data is retrieved, decrypted, then decompressed
$retrieved = $cache->getCache('payment');
// $retrieved === $sensitiveData
```
