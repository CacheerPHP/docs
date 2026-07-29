# API Reference

Complete reference for the CacheerPHP 6.x public API. v6 is instance-first: a
small `Cache` kernel over a minimal `Store` contract, with optional capabilities
and composable decorators.

## Core

| Page | Description |
|------|-------------|
| [Overview](./overview.md) | Architecture, namespaces, and how the pieces fit |
| [Cache & ScopedCache](./cache-functions.md) | `get`, `set`, `remember`, `flexible`, `scope`, batch, and named constructors |
| [Value objects](./option-builder.md) | `Key`, `Scope`, `Ttl`, `CacheEntry` |
| [TTL & Clock](./time-builder.md) | TTL inputs, forever, and the injected `Clock` |

## Stores

| Page | Description |
|------|-------------|
| [Stores & capabilities](./drivers.md) | The `Store` contract, capability interfaces, built-in stores, and decorators |
| [Configuration](./config.md) | Named constructors and the `PipelineConfig` storage pipeline |
| [Compression & encryption](./compression-encryption.md) | The envelope, gzip, and authenticated AES-256-GCM |
| [Locks](./locks.md) | `LockingStore`, `Lock`, and single-flight coordination |

## Interop

| Page | Description |
|------|-------------|
| [PSR-16 & PSR-6 adapters](./psr16-adapter.md) | `Psr16Cache`, `Psr6Pool`, `Psr6Item` |

## Conventions

- **Instance-first.** There is no global state. Construct a `Cache` and pass it
  where you need it.
- **Keys are strings or `Key` objects.** Every method that takes a key accepts
  both; a bare string is wrapped as `Key::named($string)`.
- **Time is injected.** Every constructor accepts an optional `Clock`; the
  default is `SystemClock`. Tests use `FakeClock`.
- **Capabilities are checked, never faked.** Calling a capability a store does not
  implement throws `UnsupportedCapabilityException` instead of degrading silently.
- **Values round-trip losslessly**, including `null`, `false`, `0`, `''`, and
  `[]`. A cached `null` is a hit, not a miss — use `Cache::entry()` to tell them
  apart.

See also the [migration guide](../updating/index.md) and the runnable, CI-tested
examples in the package's `examples/v6` directory.
