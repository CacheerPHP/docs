# Releases and version support

Choose the documentation that matches the version installed in your application.

## Current release lines

| Line | Status | PHP | Installation |
|---|---|---|---|
| 6.x | Current stable line; starts at 6.0.0 | 8.3+ | `composer require silviooosilva/cacheer-php:"^6.0"` |
| 5.x | Maintenance; latest release is 5.2.0 | 8.2+ | `composer require silviooosilva/cacheer-php:"^5.2"` |

CacheerPHP 6.0.0 is the stable release for the instance-first API documented here.
The `^6.0` constraint accepts stable 6.x updates without moving to another major version. See [GitHub releases](https://github.com/CacheerPHP/CacheerPHP/releases) for release notes and changelogs.

## Support during migration

The [v6 migration guide](./index.md#support-window) sets the support window: v6 receives features, correctness fixes, and security updates. v5 receives security and correctness fixes only for 12 months from the 6.0.0 release; new features are not backported.

Use the [v5 documentation](../../../v5/en/getting-started/index.md) for an application still running v5. Follow the [migration guide](./index.md) to upgrade to the stable 6.x line.

## Reporting a problem

Open an [issue](https://github.com/CacheerPHP/CacheerPHP/issues) with your CacheerPHP and PHP versions, selected store, a small reproduction, and the expected behavior. For Monitor integration issues, include the output of `vendor/bin/cacheer-monitor doctor`.

The project is maintained by [Silvio Silva](https://github.com/silviooosilva). Contributions are described in the [contribution guide](../contributing/index.md).
