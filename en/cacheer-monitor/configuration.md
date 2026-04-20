# Configuration & Environment

The monitor favours convention, yet a few knobs keep it flexible during development.

## Events File Resolution

The monitor resolves the events file in this order:

1. **OS environment** — `CACHEER_MONITOR_EVENTS` set in the shell or process env (origin: `env`)
2. **`.env` file** — read from the project root via `src/Support/Env.php` (origin: `dotenv`)
3. **Fallback** — `sys_get_temp_dir()/cacheer-monitor.jsonl` (origin: `default`)

Relative paths are resolved against the consuming project root, never the monitor package directory under `vendor/`.

## Environment Variables

All variables can be set in a `.env` file at the project root or as real OS environment variables. Use `.env.example` from the monitor package as a starting point:

```bash
cp vendor/cacheerphp/monitor/.env.example .env
```

| Variable | Default | Description |
|---|---|---|
| `CACHEER_MONITOR_EVENTS` | *(temp dir)* | Absolute or project-root-relative path to the JSONL events file. |
| `CACHEER_MONITOR_TOKEN` | — | When set, destructive API actions (`clear`, `cleanup-rotated`) require this value in the `X-Monitor-Token` request header. |
| `CACHEER_MONITOR_CAPTURE_VALUES` | `false` | When `true`, the instrumentation layer records a preview of cached values in each event payload. See [Value Capture](#value-capture) below. |
| `CACHEER_MONITOR_STREAM_TIMEOUT` | `30` | How long (seconds) the SSE `/api/events/stream` connection stays open before the client must reconnect. |
| `CACHEER_MONITOR_PREVIEW_BYTES` | `2048` | Maximum size (bytes) of the serialised value preview written into each event. Values larger than this are truncated. |
| `CACHEER_MONITOR_REDACT_KEYS` | — | Comma-separated list of additional field names to redact in value previews. Built-in redacted keys: `password`, `passwd`, `pwd`, `secret`, `token`, `access_token`, `refresh_token`, `authorization`, `api_key`, `apikey`, `private_key`, `client_secret`, `cookie`, `session`. |

## Refresh Rate

- The UI refresh interval defaults to **5 s** (adjustable via the select in the header).
- Manual refresh is available using the dashboard **Refresh** button.
- The event stream uses SSE ping fallbacks when supported.
- For large files, consider pruning via `POST /api/events/clear` after archiving.

## Value Capture

When `CACHEER_MONITOR_CAPTURE_VALUES=true`, each cache event includes a `value_preview` field containing a JSON-encoded snapshot of the cached value (up to `CACHEER_MONITOR_PREVIEW_BYTES` bytes). This powers the Key Inspector panel's value preview.

Sensitive fields are automatically redacted from previews. Extend the built-in redaction list with `CACHEER_MONITOR_REDACT_KEYS`:

```env
CACHEER_MONITOR_CAPTURE_VALUES=true
CACHEER_MONITOR_REDACT_KEYS=ssn,credit_card,dob
```

> **Note:** Value capture stores data in the JSONL log. Avoid enabling it in production or on files with long retention.

## API Token Protection

Set `CACHEER_MONITOR_TOKEN` to protect destructive endpoints:

```env
CACHEER_MONITOR_TOKEN=my-local-secret
```

Pass the token as an HTTP header when calling `POST /api/events/clear` or `POST /api/events/cleanup-rotated`:

```bash
curl -X POST http://127.0.0.1:9966/api/events/clear \
  -H "X-Monitor-Token: my-local-secret"
```

## Event Payload Format

Each line in the JSONL file is a single Cacheer event:

```json
{
  "type": "hit",
  "ts": 1714310556,
  "payload": {
    "key": "users:42",
    "namespace": "default",
    "driver": "FileCacheStore",
    "size_bytes": 2048,
    "duration_ms": 1.9,
    "ttl": 86400,
    "value_preview": "{\"id\":42,\"name\":\"Alice\"}"
  }
}
```

Additional fields (`ttl`, `value_preview`, `value_type`, `size_bytes`, `exists`, `count`) appear depending on the event type and enabled features.
