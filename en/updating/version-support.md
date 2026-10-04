# Releases and version support

Choose the documentation that matches the version installed in your application.

## Current release lines

| Line | Status | PHP | Installation |
|---|---|---|---|
| 6.x | Release candidate (6.0.0-RC); the instance-first API documented here | 8.3+ | `composer require silviooosilva/cacheer-php:"^6.0@RC"` |
| 5.x | Published stable line; latest release is 5.2.0 | 8.2+ | `composer require silviooosilva/cacheer-php:"^5.2"` |

The latest published stable release is [v5.2.0, released July 6, 2026](https://github.com/CacheerPHP/CacheerPHP/releases/tag/v5.2.0).
Release status was checked on October 3, 2026. See [GitHub releases](https://github.com/CacheerPHP/CacheerPHP/releases) for newer releases and their changelogs.

The 6.x development branch can change before a stable release. Its explicit Composer constraint is required to try the API shown in this documentation; an unversioned install currently selects the stable 5.x package.

## Support during migration

The [v6 migration guide](./index.md#support-window) sets the support window: v6 is the actively developed line, and v5 receives security and correctness fixes for 12 months after the 6.0 stable release. That window starts when 6.0 stable is released.

Use the [v5 documentation](../../../v5/en/getting-started/index.md) for an existing stable application. Use the [migration guide](./index.md) to prepare its upgrade to the instance API.

## Reporting a problem

Open an [issue](https://github.com/CacheerPHP/CacheerPHP/issues) with your CacheerPHP and PHP versions, selected store, a small reproduction, and the expected behavior. For Monitor integration issues, include the output of `vendor/bin/cacheer-monitor doctor`.

The project is maintained by [Silvio Silva](https://github.com/silviooosilva). Contributions are described in the [contribution guide](../contributing/index.md).
