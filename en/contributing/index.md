# Contributing Guide

Thanks for your interest in improving CacheerPHP! This guide explains how to get set up locally, follow the conventions used across the project, and submit your contribution smoothly.

## Prerequisites

- PHP 8.1 or newer
- Composer 2+
- A Redis server if you want to exercise Redis-specific features (optional)
- Node.js/npm if you plan to update the documentation site assets

## Repository Structure

```
CacheerPHP-Org/
├── CacheerPHP/            # Core library
│   ├── src/
│   ├── tests/
│   └── ...
├── cacheerphp.github.io/  # Published docs site (generated from /docs)
└── docs/                  # Source Markdown (EN and PT)
```

**Important**: update documentation in the `/docs` workspace. The website (`cacheerphp.github.io/resources/docs`) is derived from that content.

## Setting Up Locally

```sh
# Clone the mono-repo (contains library and docs)
git clone https://github.com/silviooosilva/CacheerPHP-Org.git
cd CacheerPHP-Org/CacheerPHP

# Install PHP dependencies
composer install

# Run the test suite
composer test

# Optional: run static analysis & coding standards
composer lint
composer analyse
```

<!-- ## Running the Monitor (Optional)

```sh
cd cacheer-monitor
composer install
php bin/cacheer-monitor serve --host=127.0.0.1 --port=9966
```

Use `CACHEER_MONITOR_EVENTS` to control the JSONL file location.

## Branching & Commits

- Work from a feature branch (`feat/...`, `fix/...`, etc.).
- Keep commits focused; use present-tense, conventional-style messages (e.g. `feat: add redis cache manager`).
- Rebase onto the latest `main` (or `master`) before opening your pull request.

## Testing Checklist

- `composer test` — PHPUnit suite
- `composer lint` — Coding standards (PHP-CS-Fixer or similar)
- `composer analyse` — Static analysis (Psalm/Larastan, depending on config)
- Any relevant monitor scripts (`php Tests/stress_io.php`) if your changes touch aggregator/reporting logic -->

## Updating Documentation

- English content lives under `docs/en/...`
- Portuguese translations go in `docs/pt/...`
- If only one language is available, link to the other as a fallback.
- When a new page is added, update the relevant `index.md` so it becomes discoverable.

## Submitting a Pull Request

1. Push your feature branch to your fork.
2. Open a PR against `silviooosilva/CacheerPHP` (or the relevant repo).
3. Fill out the template: describe the change, testing performed, screenshots (if UI-related).
4. Be responsive to review feedback; squash or rebase if asked.

## Coding Guidelines

- Follow PSR-12 coding style unless the file already uses an alternative style.
- Type declarations are encouraged (`declare(strict_types=1);`, scalar/type hints).
- Avoid introducing new dependencies unless necessary; discuss first in an issue.
<!-- - For monitor frontend changes, keep Tailwind utility classes consistent and avoid inline styles when possible. -->

## Reporting Issues

- Use GitHub Issues.
- Provide a clear title, environment details, reproduction steps, expected vs actual behaviour.
<!-- - Attach logs, stack traces, or JSONL samples if the bug affects the monitor. -->

Thank you for helping make CacheerPHP better!
