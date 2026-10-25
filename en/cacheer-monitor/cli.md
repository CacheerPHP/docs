# CLI Reference

Helper commands live in `bin/cacheer-monitor`.

## `serve`

Starts the HTTP API and SPA assets using PHP's built-in server.

| Flag | Default | Description |
|---|---|---|
| `--host` | `127.0.0.1` | Bind address |
| `--port` | `9966` | Listen port |
| `--events` | Auto-resolved | Path to the JSONL events file. |
| `--workers` | `4` | Server worker processes on Unix; SSE holds a worker for the stream lifetime. Ignored on Windows. |
| `--quiet` | — | Suppress request logging |

```bash
vendor/bin/cacheer-monitor serve --host=127.0.0.1 --port=9966 --events=/absolute/path/to/events.jsonl --workers=4 --quiet
```

## `doctor`

Checks the autoload bridge end to end and reports why it is inactive when it is.

```bash
vendor/bin/cacheer-monitor doctor
```

```
  [ok]  CacheerPHP telemetry tap Observability\Telemetry found
  [ok]  Autoload bridge       active
  [ok]  Listener registered   caches will report
  [ok]  Events file           /tmp/cacheer-monitor.jsonl
```

Verifies that a CacheerPHP with the telemetry tap is installed, that Composer ran
the bootstrap, that a listener is registered, and that the events file is
writable. Exits non-zero when a check fails, so it works in CI.

The bridge runs at autoload in every request and so can never warn or throw —
this command is where its silence gets explained.

## `help`

```sh
vendor/bin/cacheer-monitor help
```

For package development and regression checks, see
[Contributing to Monitor](../contributing/cacheer-monitor.md).
