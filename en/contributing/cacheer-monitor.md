# Contributing to Cacheer Monitor

Open an issue or pull request in [CacheerPHP/monitor](https://github.com/CacheerPHP/monitor).
Monitor is a PHP package with a static dashboard; it has no required frontend build step.

## Setup and checks

You need PHP 8.3+, Composer, and Node.js with npm for dashboard development.

```sh
git clone https://github.com/CacheerPHP/monitor.git
cd monitor
composer install
npm ci
composer test
npm run test:ui
npm run lint
```

`composer test` runs PHPUnit. PHP behavior tests live in `Tests/Unit`, while
diagnostics and Node dashboard regressions live in `Tests/Regression`.
`npm run test:ui` runs the Node regressions; it is not a browser visual test.
`npm run lint` checks JavaScript with ESLint. There is no `npm test` script.

Use a disposable events file and free loopback port for manual checks. Point
the application reporter and server at the same file; see the
[quick start](../cacheer-monitor/quick-start.md). Keep credentials, captured
application data, and generated logs out of commits. Check dashboard changes in
both themes, at narrow widths, and with keyboard navigation.
