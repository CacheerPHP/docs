# Quick Start

Get telemetry flowing to the dashboard in under a minute.

## Install

```bash
composer require cacheerphp/monitor
```

**That's the entire integration.** The package self-registers via Composer's `autoload.files` — as soon as `vendor/autoload.php` is loaded, all cache operations on every `Cacheer` instance are instrumented automatically. No code changes required.

## Start the Dashboard

In a separate terminal from the project root:

```bash
vendor/bin/cacheer-monitor serve --port=9966
```

Open [http://127.0.0.1:9966](http://127.0.0.1:9966) in your browser and run your app normally — the dashboard updates in real time.

---

## Custom Events File Path

Events are written to the path resolved in this order:

1. `CACHEER_MONITOR_EVENTS` environment variable
2. `.env` file in the project root
3. System temp dir (`sys_get_temp_dir() . '/cacheer-monitor.jsonl'`)

To override, remove the auto-registered listener and add your own after `vendor/autoload.php`:

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Cacheer\Monitor\CacheerMonitorListener;
use Cacheer\Monitor\Reporter\JsonlReporter;

Cacheer::removeListeners();
Cacheer::addListener(new CacheerMonitorListener(
    new JsonlReporter('/var/log/myapp/cacheer-events.jsonl')
));
```

Start the server pointing to the same path:

```bash
CACHEER_MONITOR_EVENTS=/var/log/myapp/cacheer-events.jsonl \
  vendor/bin/cacheer-monitor serve --port=9966
```

---

## Multiple Listeners

You can register more than one listener — useful for sending events to different backends simultaneously:

```php
Cacheer::addListener(new CacheerMonitorListener(new JsonlReporter()));
Cacheer::addListener(new MyCustomAlerter());
```

To remove all listeners:

```php
Cacheer::removeListeners();
```

---

## Generate Sample Data

To produce a burst of events for demo or testing:

```bash
php Tests/scenarios.php
php Tests/scenarios_advanced.php   # bursty load + error cases
```

Use the dashboard **Clear** button or `DELETE /api/events/clear` to reset recorded events.

---

## Legacy: Manual Registration

Before v1.0.0, the recommended approach was to wrap the `Cacheer` instance manually or call `addListener` by hand. Both still work but are no longer needed for most projects:

```php
// Manual listener (still valid for custom config):
Cacheer::addListener(new CacheerMonitorListener(new JsonlReporter()));

// Old wrapper approach (still works, but requires code changes):
$instrumented = InstrumentedCacheer::wrap($cacheer, new JsonlReporter());
```
