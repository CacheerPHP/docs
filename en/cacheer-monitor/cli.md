# CLI Reference

Helper commands live in `bin/cacheer-monitor`.

## `serve`

Starts the HTTP API and SPA assets using PHP's built-in server.

| Flag | Default | Description |
|---|---|---|
| `--host` | `127.0.0.1` | Bind address |
| `--port` | `9966` | Listen port |
| `--quiet` | — | Suppress request logging |

```bash
php bin/cacheer-monitor serve --host=0.0.0.0 --port=9000 --quiet
```

## Scripts & Reporters

Run bundled demos to exercise Cacheer adapters and produce realistic traffic.

| Command | Description |
|---|---|
| `php Tests/scenarios.php` | Baseline traffic mix |
| `php Tests/scenarios_advanced.php` | Bursty load, tag flushing, error events |
| `php Tests/metrics_test.php` | Quick regression check for metrics aggregation |
