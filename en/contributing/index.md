# Contributing

Thanks for helping improve CacheerPHP. This page covers the v6 development
workflow. For the full policy, see `CONTRIBUTING.md`, `SECURITY.md`, and the
`ROADMAP.md` in the package repository.

## Setup

```sh
git clone https://github.com/silviooosilva/CacheerPHP
cd CacheerPHP
composer install
```

PHP 8.3+ is required. The core has no mandatory extensions; install `pdo_sqlite`,
`openssl`, and `zlib` to run the full local suite, and `predis/predis` for Redis.

## Test suites

Tests use [Pest](https://pestphp.com/) and are split by concern:

```sh
composer test            # service-free unit suite (default)
composer test:kernel     # v6 kernel (Cacheer, adapters, CLI, rehearsals)
composer test:contract   # store conformance (Array, File)
composer test:storage    # the storage pipeline / envelope
composer test:concurrency  # lock and counter contention harnesses
composer test:integration  # Redis / database (needs services)
composer test:all        # everything
```

Data providers must use the attribute form, not the annotation:

```php
#[\PHPUnit\Framework\Attributes\DataProvider('cases')]
```

## Static analysis and style

```sh
composer analyse   # PHPStan level 5, suppression-free
composer lint      # php-cs-fixer (dry-run)
composer fix       # php-cs-fixer (apply)
```

## Deterministic time

Never call `time()` or `sleep()` in tests or store code. Time flows through an
injected `Clock`; tests advance a `FakeClock`. This keeps the suite fast and
expiry/stale behavior exact.

## Adding a store

Implement the `Store` contract, add only the capabilities you can guarantee, and
prove it by extending `Tests\Support\StoreConformance`. See the
[Custom stores guide](../guides/custom-stores.md). A store must never scan or clear
data outside its configured keyspace.

## Quality gates for a pull request

- Maps to a roadmap work item (and an accepted RFC for substantial changes).
- Public behavior has unit or contract tests; driver-specific behavior has
  integration tests; concurrency claims have contention tests.
- New configuration is typed and documented; new events carry metadata only
  (never values).
- No hidden filesystem, environment, timezone, schema, or network side effects.
- `composer analyse` and `composer lint` pass.
- Relevant examples and migration notes are updated.
- Performance-sensitive changes include before/after benchmark evidence
  (`composer benchmark:baseline`).

## Reporting

- **Bugs / features:** use the issue templates in the repository.
- **Security:** do not open a public issue — follow `SECURITY.md`.
- **Substantial changes:** open an RFC first so the design is agreed before code.
