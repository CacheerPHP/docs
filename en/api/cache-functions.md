# Cache Functions — API Reference

This page documents every cache operation available on the `Cacheer` class. All methods can be called on an instance (`$cache->method()`) or statically (`Cacheer::method()`).

---

## Status Methods

After any operation you can inspect the result:

```php
$cache->isSuccess();   // bool — whether the last operation succeeded
$cache->getMessage();  // string — human-readable status message
```

---

## Reading Data

### `getCache()` — Retrieve a single item

```php
$cache->getCache(string $cacheKey, string $namespace = '', string|int $ttl = 3600): mixed
```

Returns the cached value, or `null` if the key does not exist or has expired. In v5.0.0, falsy values (`0`, `false`, `''`, `[]`) are correctly returned as cache hits — check `isSuccess()` to distinguish a stored falsy value from a miss.

```php
$user = $cache->getCache('user:123');

if ($cache->isSuccess()) {
    // Cache hit — $user may be any value including 0, false, or ''
} else {
    // Cache miss
}
```

### `getMany()` — Retrieve multiple items

```php
$cache->getMany(array $cacheKeys, string $namespace = ''): array
```

Returns an associative array `[key => value]` for each found key.

```php
$items = $cache->getMany(['key1', 'key2', 'key3']);
// ['key1' => 'val1', 'key2' => 'val2', 'key3' => 'val3']
```

### `getAll()` — Retrieve all items in a namespace

```php
$cache->getAll(string $namespace = ''): array
```

Returns every cached item within the given namespace.

### `has()` — Check key existence

```php
$cache->has(string $cacheKey, string $namespace = ''): bool
```

Returns `true` if the key exists and is not expired.

```php
if ($cache->has('session:abc')) {
    // Key exists
}
```

---

## Writing Data

### `putCache()` — Store a value

```php
$cache->putCache(
    string $cacheKey,
    mixed  $cacheData,
    string $namespace = '',
    int|string|\DateInterval|null $ttl = 3600
): bool
```

Stores a value under the given key. Returns `true` on success. The `$ttl` parameter accepts:

| Type | Meaning | Example |
|------|---------|---------|
| `int` | Seconds | `3600` |
| `string` | Human-readable | `'2 hours'` |
| `\DateInterval` | PHP interval *(new in v5)* | `new \DateInterval('PT30M')` |
| `null` | Store forever *(new in v5)* | `null` |

```php
$cache->putCache('key', $data, '', 3600);
$cache->putCache('key', $data, '', '2 hours');
$cache->putCache('key', $data, '', new \DateInterval('PT30M'));
$cache->putCache('key', $data, '', null);  // forever
```

### `putMany()` — Store multiple items

```php
$cache->putMany(array $items, string $namespace = '', int $batchSize = 100): bool
```

Stores an array of items in batch. Each element must have `cacheKey` and `cacheData`.

### `appendCache()` — Append to existing value

```php
$cache->appendCache(string $cacheKey, mixed $cacheData, string $namespace = ''): bool
```

Replaces (appends) data to an existing cache entry.

### `forever()` — Store without expiration

```php
$cache->forever(string $cacheKey, mixed $cacheData): bool
```

Stores a value with `PHP_INT_MAX` as the TTL — effectively no expiration.

### `add()` — Conditional store (only if key is absent)

```php
$cache->add(
    string $cacheKey,
    mixed  $cacheData,
    string $namespace = '',
    int|string|\DateInterval|null $ttl = 3600
): bool
```

Stores the value **only if the key does not already exist**.

| Return | Meaning |
|--------|---------|
| `true` | Key was new — value was stored |
| `false` | Key already existed — nothing was written |

> **v5 breaking change:** In v4.x the return values were inverted. v5.0.0 matches the standard convention used by memcached, Laravel, and other mainstream libraries.

```php
// First-writer-wins lock pattern
if ($cache->add('lock:job:42', getmypid(), ttl: 60)) {
    // Lock acquired — run the job
    $cache->clearCache('lock:job:42');
} else {
    // Another process holds the lock
}
```

---

## Modifying Data

### `renewCache()` — Extend TTL without changing data

```php
$cache->renewCache(
    string $cacheKey,
    int|string|\DateInterval|null $ttl = 3600,
    string $namespace = ''
): bool
```

### `increment()` / `decrement()` — Numeric operations

```php
$cache->increment(string $cacheKey, int $amount = 1, string $namespace = ''): bool
$cache->decrement(string $cacheKey, int $amount = 1, string $namespace = ''): bool
```

Increments or decrements a numeric cached value. Returns `true` on success. In v5.0.0, these correctly handle stored value `0` (which was treated as a miss in v4).

```php
$cache->putCache('counter', 0);
$cache->increment('counter', 5);   // counter is now 5
$cache->decrement('counter', 2);   // counter is now 3
```

---

## Computed Values

### `remember()` — Get-or-compute with TTL

```php
$cache->remember(
    string $cacheKey,
    int|string|\DateInterval $ttl,
    \Closure $callback
): mixed
```

Returns the cached value if it exists; otherwise executes `$callback`, stores the result, and returns it. In v5.0.0, the callback is **not** re-invoked when the cached value is falsy (`0`, `false`, etc.).

```php
$stats = $cache->remember('dashboard:stats', new \DateInterval('PT5M'), function () {
    return computeExpensiveStats();
});
```

### `rememberForever()` — Get-or-compute without expiration

```php
$cache->rememberForever(string $cacheKey, \Closure $callback): mixed
```

Same as `remember()`, but stored with no expiration.

### `getAndForget()` — Retrieve and remove

```php
$cache->getAndForget(string $cacheKey, string $namespace = ''): mixed
```

Returns the cached value and immediately deletes the key. Returns `null` if the key does not exist.

---

## Removing Data

### `clearCache()` — Remove a single key

```php
$cache->clearCache(string $cacheKey, string $namespace = ''): bool
```

### `flushCache()` — Remove everything

```php
$cache->flushCache(): bool
```

Deletes all cached items from the current driver.

---

## Tagging

### `tag()` — Associate keys with a tag

```php
$cache->tag(string $tag, string ...$keys): bool
```

Groups one or more keys under a tag name. Keys can be plain (`'user:1'`) or namespaced (`'nsA:profile'`).

### `flushTag()` — Invalidate by tag

```php
$cache->flushTag(string $tag): bool
```

Removes all keys associated with the given tag.

```php
$cache->putCache('user:1', ['name' => 'Alice']);
$cache->putCache('user:2', ['name' => 'Bob']);
$cache->tag('users', 'user:1', 'user:2');

$cache->flushTag('users');  // removes user:1 and user:2
```

**Storage by driver:**

| Driver | Where the tag index lives |
|--------|--------------------------|
| File | `cacheDir/_tags/{tag}.json` |
| Redis | Redis Set `tag:{tag}` |
| Database | Reserved namespace `__tags__`, key `tag:{tag}` |
| Array | In memory; reset on `flushCache()` |

---

## Compression & Encryption

### `useCompression()` — Toggle gzip compression

```php
$cache->useCompression(bool $enable = true): self
```

When enabled, data is serialized and compressed with `gzcompress` before storage.

### `useEncryption()` — Enable AES-256-CBC encryption

```php
$cache->useEncryption(string $key): self
```

Encrypts data with AES-256-CBC using a random IV per write (v5.0.0). The IV is prepended to the ciphertext and stored as a base64-encoded blob.

Both features can be combined:

```php
$cache->useCompression()->useEncryption('my-secret-key');
```

See [Compression & Encryption](./compression-encryption.md) for details.

---

## Diagnostics (v5.0.0)

### `stats()` — Inspect the active instance

```php
$cache->stats(): array
```

Returns:

```php
[
    'driver'      => 'Silviooosilva\CacheerPhp\CacheStore\ArrayCacheStore',
    'compression' => false,
    'encryption'  => true,
]
```

### `getCacheStore()` — Access the active driver

```php
$cache->getCacheStore(): CacheerInterface
```

### `resetInstance()` — Clear the static singleton

```php
Cacheer::resetInstance(): void
```

Clears the shared static instance so the next static call creates a fresh one. Useful for test isolation.

### `setInstance()` — Inject a custom singleton

```php
Cacheer::setInstance(Cacheer $instance): void
```

Replaces the static singleton so all subsequent `Cacheer::*()` calls delegate to the given instance.

---

## PSR-16 Adapter

CacheerPHP ships with a PSR-16 adapter. See [PSR-16 Adapter](./psr16-adapter.md) for the full reference.

```php
use Silviooosilva\CacheerPhp\Psr\Psr16CacheAdapter;

$psr = new Psr16CacheAdapter($cache, 'optional-namespace');
$psr->set('key', 'value', 3600);
$psr->get('key');
```
