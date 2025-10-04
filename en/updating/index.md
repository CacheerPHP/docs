# Updating CacheerPHP

This guide walks you through upgrading CacheerPHP while keeping existing projects stable.

## 1. Review the changelog

Before changing dependencies, check:

- The release notes on GitHub
- `CHANGELOG.md` inside the package
- Breaking changes highlighted in pull requests

## 2. Update the library

```sh
composer update silviooosilva/cacheer-php
```

If you pin versions in `composer.json`, bump the constraint first (for example `^2.4` → `^2.5`).

After updating, clear Composer’s cache if you run into lock file conflicts:

```sh
composer clear-cache
rm composer.lock
composer update
```

## 3. Run the test suite

Always verify your project:

- `composer test`
- `composer lint`
- `composer analyse`

<!-- If your project uses the Monitor, replay the synthetic scenarios:

```sh
php cacheer-monitor/Tests/stress_test.php
php cacheer-monitor/Tests/stress_io.php
```
-->

## 4. Refresh configuration & assets

- Re-run your configuration scripts if new env vars were introduced
- Clear cache directories (`rm -rf storage/cache` or similar)
<!-- - Rebuild frontend bundles (if you bundle the monitor with your app) -->

<!--
## 5. Update Cacheer Monitor

Inside `cacheer-monitor/`:

```sh
composer install
php bin/cacheer-monitor serve --host=127.0.0.1 --port=9966
```

The Monitor CLI shares the same semantic version as the library; keep them aligned.
-->

## 6. Verify integrations

- Applications using Redis: ensure new commands are available
- Framework integrations: run feature tests (Laravel, Symfony, etc.)
- Custom reporters: re-run logging and queue pipelines

## 7. Rollback plan

Keep the previous lock file and vendor directory until you are confident the new version works. If something fails:

```sh
git checkout composer.json composer.lock
composer install
```

## 8. Report issues

If you hit problems, open an issue with:

- Version before and after
- OS/PHP version
- Stack trace or failing test case
- Steps to reproduce

Up-to-date projects benefit from the latest optimisations and security patches—thanks for keeping CacheerPHP current!
