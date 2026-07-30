# TTL & Clock

> v5's `TimeBuilder` fluent API is replaced by the [`Ttl`](./option-builder.md#ttl)
> value object and an injected `Clock`.

## Accepted TTL inputs

Anywhere a TTL is expected (`set`, `remember`, `setMany`, policies) you can pass:

```php
$cache->set('k', $v, ttl: 3600);                        // int — seconds
$cache->set('k', $v, ttl: '2 hours');                   // human string
$cache->set('k', $v, ttl: new DateInterval('PT30M'));   // DateInterval
$cache->set('k', $v, ttl: Ttl::minutes(30));            // Ttl object
$cache->set('k', $v, ttl: null);                        // forever
$cache->set('k', $v, ttl: 'forever');                   // forever (string)
```

Human strings match `<n> second|minute|hour|day|week` (singular or plural), plus
the literal `forever`. A bare integer string (`'3600'`) is treated as seconds.
Anything else throws `InvalidTtlException`.

## The `Ttl` object

```php
use Silviooosilva\CacheerPhp\Kernel\Ttl;

Ttl::seconds(30);   Ttl::minutes(10);   Ttl::hours(2);
Ttl::days(7);       Ttl::weeks(2);      Ttl::forever();
Ttl::until(new DateTimeImmutable('2026-01-01'), $clock);
```

- `seconds()` (and its multiples) require a value **greater than zero** —
  `InvalidTtlException` otherwise. There is no "zero TTL means delete" rule;
  deletion is explicit.
- Overflow is guarded: a duration or expiry that would exceed `PHP_INT_MAX`
  throws rather than silently wrapping. Forever is represented as "no expiry",
  not as a huge integer, so it does not depend on `PHP_INT_MAX` arithmetic.

## The `Clock`

Every time read goes through a `Clock`, so behavior is deterministic and testable.

```php
namespace Silviooosilva\CacheerPhp\Contracts;

interface Clock
{
    public function now(): int;          // unix seconds
    public function nowFloat(): float;   // sub-second precision
    public function sleep(int $microseconds): void;
}
```

- **Production:** `Silviooosilva\CacheerPhp\Support\SystemClock`.
- **Tests:** a fake clock you can advance by hand — no `sleep()` needed.

```php
use Silviooosilva\CacheerPhp\Support\SystemClock;

$cache = Cacheer::file('/var/cache', clock: new SystemClock());
```

Because the clock is injected everywhere (stores, decorators, policies, locks,
PSR adapters), you can freeze or fast-forward time in tests to hit exact expiry
and stale-window boundaries. See the [Custom stores guide](../guides/custom-stores.md)
for how a store must use the clock instead of `time()`.
