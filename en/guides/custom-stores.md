# Writing a custom store

You can back CacheerPHP with any storage engine without touching the kernel or
reading the built-in store source. Implement the four-method `Store` contract, add
capability interfaces for anything extra you support, and prove it with the shared
conformance suite.

## 1. The minimal store

```php
use Silviooosilva\CacheerPhp\Contracts\{Store, Clock};
use Silviooosilva\CacheerPhp\Kernel\{CacheEntry, Key, Ttl};

final class MyStore implements Store
{
    public function __construct(private readonly Clock $clock) {}

    public function get(Key $key): CacheEntry
    {
        // look up by $key->identity(); on a live hit:
        //   return CacheEntry::hit($key, $value, $createdAt, $expiresAt); // $expiresAt null = forever
        return CacheEntry::miss($key);
    }

    public function set(Key $key, mixed $value, Ttl $ttl): void
    {
        $expiresAt = $ttl->expiresAt($this->clock); // null = forever
        // persist...
    }

    public function delete(Key $key): bool { /* true if something was removed */ }

    public function clear(): void { /* only your configured keyspace */ }
}
```

Rules the conformance suite enforces:

- **Time comes from the injected `Clock`** — never call `time()` directly. That is
  what makes behavior testable with a fake clock.
- **Expiry is lazy.** `get()` on an expired entry returns a miss.
- **Values round-trip losslessly**, including `false`, `0`, `''`, `[]`, and nested
  structures. Use the storage pipeline rather than bespoke serialization.
- **`clear()` and every scan stay inside your keyspace** — never touch unrelated
  data in a shared backend.

## 2. Add only the capabilities you can honor

Implement the [capability interfaces](../api/drivers.md#capability-interfaces) you
can actually guarantee. The kernel throws `UnsupportedCapabilityException` for the
rest instead of pretending.

## 3. Reuse the storage pipeline

Encode values through an `EnvelopeCodec` from a [`PipelineConfig`](../api/config.md)
to get serialization, optional compression, authenticated encryption, size limits,
and v5 read compatibility for free:

```php
$codec = PipelineConfig::default()->withGzip()->codec();
$blob  = $codec->encode($value);   // versioned envelope
$value = $codec->decode($blob);    // typed failure on tampering / over-limit
```

## 4. Prove it with the conformance suite

Extend `Tests\Support\StoreConformance` and return your store from
`createStore()`. It runs the full base contract plus every capability block your
store declares, and skips the ones it doesn't:

```php
use Tests\Support\{FakeClock, StoreConformance};
use Silviooosilva\CacheerPhp\Contracts\Store;

final class MyStoreConformanceTest extends StoreConformance
{
    protected function createStore(FakeClock $clock): Store
    {
        return new MyStore($clock);
    }
}
```

If it passes, your store composes with `Cache`, scopes, tiering, resilience, the
PSR adapters, and the CLI exactly like the built-ins.

## 5. Getting listed as compatible

A community adapter is listed as compatible when it passes the conformance suite in
CI on supported PHP versions, documents which capabilities it provides and their
guarantees, and documents its failure modes (connection loss, timeouts). See the
`WRITING_A_STORE.md` guide and the driver-proposal issue template in the package
repository.
