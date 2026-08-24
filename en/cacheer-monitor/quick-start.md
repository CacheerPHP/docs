# Quick Start

Get telemetry flowing to the dashboard in under a minute.

## Install

```bash
composer require cacheerphp/monitor
```

**That's the entire integration.** The package self-registers via Composer's `autoload.files` — as soon as `vendor/autoload.php` is loaded, it registers a listener on CacheerPHP 6's global telemetry tap and every cache reports automatically. No code changes required.

That covers every way a cache can be built: the named constructors, `Cacheer::build()`, a plain `new Cacheer($store)`, and the `tiered()` / `resilient()` decorators. Scoped and policy-bound views inherit it, and capability operations (`increment`, `decrement`, `touch`, `tag`, `flushTag`) report too.

> Requires **CacheerPHP 6**. Until 6.0 is tagged, the dependency must allow the dev branch (`^6.0@dev`); see [when nothing shows up](#when-nothing-shows-up).

## Start the Dashboard

In a separate terminal from the project root:

```bash
vendor/bin/cacheer-monitor serve --port=9966
```

Open [http://127.0.0.1:9966](http://127.0.0.1:9966) in your browser and run your app normally — the dashboard updates in real time.

> **Note:** If your project doesn't have a `.env` file, the monitor will still work — events will be stored in the system temp directory. For a persistent, predictable path, create a `.env` at your project root. You can use the one from the CacheerPHP package as a starting point:
>
> ```bash
> cp vendor/silviooosilva/cacheer-php/.env.example .env
> ```
>
> Then add the following line to set the events file path:
>
> ```env
> CACHEER_MONITOR_EVENTS=/your/path/cacheer-monitor.jsonl
> ```

---

## Custom Events File Path

Events are written to the path resolved in this order:

1. `CACHEER_MONITOR_EVENTS` environment variable
2. `.env` file in the project root
3. System temp dir (`sys_get_temp_dir() . '/cacheer-monitor.jsonl'`)

The simplest override needs no code:

```bash
CACHEER_MONITOR_EVENTS=/var/log/myapp/cacheer-events.jsonl
```

Start the server pointing at the same path:

```bash
CACHEER_MONITOR_EVENTS=/var/log/myapp/cacheer-events.jsonl \
  vendor/bin/cacheer-monitor serve --port=9966
```

---

## Wiring a listener yourself

Turn the bridge off first, or you will have two listeners writing:

```env
CACHEER_MONITOR_AUTO_REGISTER=false
```

```php
use Cacheer\Monitor\CacheerMonitorListener;
use Cacheer\Monitor\Reporter\JsonlReporter;
use Silviooosilva\CacheerPhp\Observability\Telemetry;

Telemetry::listen(
    (new CacheerMonitorListener(new JsonlReporter('/var/log/myapp/cacheer-events.jsonl')))->dispatch(...)
);
```

## Multiple listeners

The tap fans out, so register as many as you like — useful for sending events to
several backends at once:

```php
Telemetry::listen((new CacheerMonitorListener(new JsonlReporter()))->dispatch(...));
Telemetry::listen($myCustomAlerter->handle(...));
```

A listener that throws can never break a cache operation; the tap swallows the
failure. To drop every listener (including the auto-registered one):

```php
Telemetry::reset();
```

## Instrumenting only one cache

The bridge is deliberately all-or-nothing. To instrument a single cache instead,
disable it and use CacheerPHP's own `instrumented()` constructor:

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Observability\EventBus;

$events = new EventBus();
$events->listen((new CacheerMonitorListener(new JsonlReporter()))->dispatch(...));

$cache = Cacheer::instrumented($store, $events);   // only this one reports
```

---

## When nothing shows up

The bridge runs at autoload in every request, so it can never warn or throw —
silence is its only safe failure mode. `doctor` is where that silence gets
explained:

```bash
vendor/bin/cacheer-monitor doctor
```

```
  [ok]  CacheerPHP telemetry tap Observability\Telemetry found
  [ok]  Autoload bridge       active
  [ok]  Listener registered   caches will report
  [ok]  Events file           /tmp/cacheer-monitor.jsonl
```

It exits non-zero when a check fails, so it works in CI. The two usual causes:

- **The events file.** With no `CACHEER_MONITOR_EVENTS` set it defaults to the
  system temp directory, so "nothing is reported" is often "the dashboard is
  reading a different file".
- **The CacheerPHP version.** The monitor hooks CacheerPHP 6's telemetry tap,
  which does not exist in v4/v5. Until 6.0 is tagged the constraint must allow
  the dev branch (`^6.0@dev`) — a plain `^6.0` silently fails
  `minimum-stability: stable` and leaves an old release installed with no tap to
  hook.
