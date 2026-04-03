# Cacheer Monitor

Cacheer Monitor ships with the CacheerPHP repository and provides a real-time dashboard for visualising cache activity. It reads JSONL events emitted by the `JsonlReporter` and renders metrics, timelines, namespace activity and the raw event stream in a responsive SPA.

## Requirements

- PHP 8.1+
- Composer dependencies installed inside `cacheer-monitor`
- Optional: Redis or other Cacheer drivers if you want to showcase mixed driver traffic

Install the dependencies the first time you clone the repository:

```sh
cd cacheer-monitor
composer install
```

## Start the Dashboard

Launch the built-in PHP server:

```sh
php bin/cacheer-monitor serve --host=127.0.0.1 --port=9966
```

The dashboard lives at <http://127.0.0.1:9966>. Use `--host=0.0.0.0` to expose it on your LAN and `--port` to change the default port.

By default the monitor logs to a temp JSONL file. Override the path before starting the server:

```sh
export CACHEER_MONITOR_EVENTS=/path/to/cacheer-monitor.jsonl
```

## Feed the Monitor

Three scripts generate rich workloads:

```sh
php Tests/scenarios.php            # baseline mix of hits, misses, puts, tags
php Tests/stress_io.php            # I/O heavy scenario: multiple namespaces, drivers, errors
php Tests/stress_test.php          # synthetic load validating aggregator math
```

All of them honour `CACHEER_MONITOR_EVENTS`, so use the same path consumed by the dashboard.

## UI Overview

- **Metrics bar** – cards with hits, misses, puts, flushes, tags, errors and latency percentiles.
- **Recent events** – newest entries list event type, namespace, driver, TTL, duration and payload size.
- **Drivers & top keys** – collapsible sections highlighting driver share, namespace volume and hot keys.
- **Timeline insights** – charts for hits vs misses and latency evolution over the last 10 minutes.

Adjust the auto-refresh selector in the header or pause it to refresh manually.

## REST API

The SPA talks to lightweight endpoints you can also script against:

```http
GET  /api/health            # liveness probe → { "ok": true }
GET  /api/config            # resolved events file + origin metadata
GET  /api/metrics           # counters, latency, drivers, namespaces, top keys
GET  /api/events            # recent events with optional limit & namespace filters
POST /api/events/clear      # rotates the JSONL file
GET  /api/events/stream     # SSE heartbeat for push refreshes
```

## CLI Helpers

`bin/cacheer-monitor` currently exposes:

- `serve` — starts the local server (`--host`, `--port`, `--quiet` to suppress server logs)
- more commands will arrive as automation hooks are added

Environment variables (`CACHEER_MONITOR_EVENTS`, `CACHEER_MONITOR_HOST`, etc.) are respected by every command.

## Configuration Cheatsheet

- `CACHEER_MONITOR_EVENTS`: absolute/relative path to the JSONL log (relative paths resolve from repo root).
- `CACHEER_MONITOR_HOST` / `CACHEER_MONITOR_PORT`: defaults used by the `serve` command.
- `CACHEER_MONITOR_REFRESH`: initial polling interval in ms (`off` disables auto-refresh).

## Suggested Workflows

- **Fresh demo**: `POST /api/events/clear`, run `php Tests/scenarios.php`, set refresh to 1s, showcase the UI.
- **Latency investigation**: filter by namespace/key and focus on the latency badges in the event list.
- **Driver comparison**: replay `Tests/stress_io.php` to highlight mixed driver usage (array, file, redis).
- **CI sanity**: execute `php Tests/stress_test.php` to ensure aggregation logic still matches expectations.

## Troubleshooting

- **Empty dashboard**: check `GET /api/config` for the resolved file path and filesystem permissions.
- **Slow UI**: trim the log via `POST /api/events/clear` when the JSONL file grows large.
- **Theme stuck**: clear browser storage if the theme toggle ignores your system preference.
- **Missing drivers**: stress scripts fall back to the array driver if Redis/File are unavailable—inspect server logs.

For deeper dives inspect `cacheer-monitor/public/assets/js` (frontend behaviour) and `cacheer-monitor/src` (HTTP API, CLI and reporters).
