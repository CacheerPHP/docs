# Example 16 — Monitor Integration *(v5.0.0)*

Add real-time telemetry to an existing CacheerPHP project with a single command — no code changes required.

## Install

```bash
composer require cacheerphp/monitor
```

The package self-registers via Composer's `autoload.files`. From this point on, every cache operation is instrumented automatically.

## Existing Code — Unchanged

```php
require 'vendor/autoload.php'; // bootstrap.php runs here — monitor is active

use Silviooosilva\CacheerPhp\Cacheer;

$cacheer = new Cacheer(['cacheDir' => __DIR__ . '/cache']);
$cacheer->setDriver()->useFileDriver();

// All of these emit telemetry events automatically:
$cacheer->putCache('user:1', ['name' => 'Alice']);
$cacheer->getCache('user:1');           // → 'hit' event
$cacheer->getCache('user:99');          // → 'miss' event
$cacheer->increment('page_views');
$cacheer->clearCache('user:1');
$cacheer->flushCache();                 // → 'flush' event

// Static facade also fires events:
Cacheer::putCache('config:locale', 'en_US');
Cacheer::getCache('config:locale');     // → 'hit' event
```

## View the Dashboard

```bash
vendor/bin/cacheer-monitor serve --port=9966
# → http://127.0.0.1:9966
```

## Custom File Path

If you need a custom events file, override after autoload:

```php
use Cacheer\Monitor\CacheerMonitorListener;
use Cacheer\Monitor\Reporter\JsonlReporter;

Cacheer::removeListeners();
Cacheer::addListener(new CacheerMonitorListener(
    new JsonlReporter('/var/log/myapp/events.jsonl')
));
```

## Events Emitted

| Operation | Event type |
|---|---|
| `getCache` (found) | `hit` |
| `getCache` (not found) | `miss` |
| `putCache` | `put` |
| `putMany` | `put_many` |
| `clearCache` | `clear` |
| `flushCache` | `flush` |
| `increment` / `decrement` | `increment` / `decrement` |
| `add` | `add` |
| `remember` / `rememberForever` | `remember` / `remember_forever` |
| `forever` | `put_forever` |
| `renewCache` | `renew` |
| `tag` / `flushTag` | `tag` / `flush_tag` |
| `has` | `has` |
| `getAndForget` | `get_and_forget` |

Each event includes: `key`, `namespace` (when non-empty), `driver`, `duration_ms`, and `success`.

## See Also

- [Cacheer Monitor — Quick Start](../cacheer-monitor/quick-start.md)
- [Cacheer Monitor — REST API](../cacheer-monitor/api.md)
- [Example 15 — Stats and Instance Management](./example-15-stats-instance.md)
