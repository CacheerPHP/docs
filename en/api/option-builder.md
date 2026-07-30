# Value objects — Key, Scope, Ttl, CacheEntry

> v5's `OptionBuilder` is gone. v6 configures stores through
> [named constructors](./cache-functions.md#named-constructors) and a typed
> [`PipelineConfig`](./config.md). The typed value objects below replace the
> stringly-typed arguments v5 passed around.

All four objects live in `Silviooosilva\CacheerPhp\Kernel`. They are immutable
and validated on construction.

## `Key`

A validated cache key, optionally bound to a [`Scope`](#scope).

```php
use Silviooosilva\CacheerPhp\Kernel\Key;

Key::named(string $value): self   // throws InvalidKeyException if empty, >1024 bytes, or has control chars

$key->value(): string             // the raw name
$key->scope(): Scope              // its scope (root by default)
$key->within(Scope $scope): self  // a copy bound to $scope
$key->identity(): string          // collision-free internal identity
```

Every `Cacheer` method accepts a plain string too; it is wrapped as
`Key::named($string)` in the caller's scope.

## `Scope`

An isolated keyspace. Scopes are ordered segment lists; the root scope is empty.

```php
use Silviooosilva\CacheerPhp\Kernel\Scope;

Scope::root(): self
Scope::named(string $segment): self
Scope::fromSegments(iterable $segments): self

$scope->child(string $segment): self   // append one segment
$scope->append(Scope $other): self
$scope->isRoot(): bool
$scope->segments(): array
$scope->contains(Scope $other): bool   // is $other within $scope?
```

Segments cannot be empty, exceed 255 bytes, or contain slashes or control
characters (`InvalidScopeException`). A scope may hold at most 64 segments. See
the [Scopes guide](../guides/scopes.md).

## `Ttl`

A normalized time-to-live. See [TTL & Clock](./time-builder.md) for the accepted
input forms and forever semantics.

```php
use Silviooosilva\CacheerPhp\Kernel\Ttl;

Ttl::forever(): self
Ttl::seconds(int): self            // must be > 0
Ttl::minutes(int) / ::hours(int) / ::days(int) / ::weeks(int): self
Ttl::until(DateTimeInterface $when, Clock $clock): self
Ttl::from(Ttl|DateInterval|int|string|null): self  // the normalizer Cacheer uses

$ttl->isForever(): bool
$ttl->inSeconds(): ?int            // null when forever
$ttl->expiresAt(Clock $clock): ?int  // absolute unix ts, or null when forever
```

## `CacheEntry`

The result of a read. Distinguishes a **miss** from a cached `null`.

```php
$entry = $cache->entry('user:42');

$entry->isHit(): bool
$entry->isMiss(): bool
$entry->value(): mixed                 // throws CacheMissException on a miss
$entry->valueOr(mixed $default): mixed // safe accessor
$entry->createdAt(): ?int              // unix ts of the write
$entry->expiresAt(): ?int              // unix ts, or null when forever
$entry->isExpired(Clock $clock): bool
$entry->remainingTtl(Clock $clock): ?int  // seconds left, null when forever, 0 on a miss
```

```php
$entry = $cache->entry('token');
if ($entry->isHit()) {
    echo $entry->value();
    echo $entry->remainingTtl(new SystemClock()) ?? 'never expires';
}
```

A cached `null` returns `isHit() === true` and `value() === null` — something
`get()` cannot express, which is why `entry()` exists.
