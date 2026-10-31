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
| `CACHEER_MONITOR_ALLOW_ORIGIN` | Unset | Opt in to cross-origin API access for a specific origin. The bundled dashboard uses the same origin. |
| `CACHEER_MONITOR_CACHE_PATH` | Unset | Application cache directory for live FileStore previews. |
| `CACHEER_MONITOR_CACHE_TABLE` | `cacheer_store` | Application table for live DatabaseStore previews. |
| `CACHEER_MONITOR_TOKEN` | — | When set, destructive API actions (`clear`, `cleanup-rotated`) require this value in the `X-Monitor-Token` request header. |
| `CACHEER_MONITOR_CAPTURE_VALUES` | `false` | When `true`, the instrumentation layer records a preview of cached values in each event payload. See [Value Capture](#value-capture) below. |
| `CACHEER_MONITOR_AUTO_REGISTER` | `true` | Set to `false` to stop the autoload bridge registering a listener, so you can wire one yourself |
| `CACHEER_MONITOR_STREAM_TIMEOUT` | `30` | How long (seconds) the SSE `/api/events/stream` connection stays open before the client must reconnect. |
| `CACHEER_MONITOR_WORKERS` | `4` | Worker processes for the local server on Unix. A live SSE connection occupies a worker until it closes. |
| `CACHEER_MONITOR_PREVIEW_BYTES` | `2048` | Maximum size (bytes) of the serialised value preview written into each event. Values larger than this are truncated. |
| `CACHEER_MONITOR_REDACT_KEYS` | — | Comma-separated list of additional field names to redact in value previews. Built-in redacted keys: `password`, `passwd`, `pwd`, `secret`, `token`, `access_token`, `refresh_token`, `authorization`, `api_key`, `apikey`, `private_key`, `client_secret`, `cookie`, `session`. |

## Refresh Rate

- The UI refresh interval defaults to **5 s** (adjustable via the select in the header).
- Manual refresh is available using the dashboard **Refresh** button.
- SSE refreshes the dashboard when new event records arrive and catches up after reconnecting. Heartbeat pings keep the stream active; they do not trigger a snapshot refresh.
- **Manual** stops interval polling. SSE can still update the dashboard when events arrive or the connection is restored.
- For large files, consider pruning via `POST /api/events/clear` after archiving.

## Value Capture

When `CACHEER_MONITOR_CAPTURE_VALUES=true`, events carrying a value can include a `value_preview` field with a JSON-encoded preview (up to `CACHEER_MONITOR_PREVIEW_BYTES` bytes). This powers the Key Inspector's preview; lifecycle markers do not carry a cached value.

Sensitive fields are automatically redacted from previews. Extend the built-in redaction list with `CACHEER_MONITOR_REDACT_KEYS`:

```env
CACHEER_MONITOR_CAPTURE_VALUES=true
CACHEER_MONITOR_REDACT_KEYS=ssn,credit_card,dob
```

> **Note:** Value capture stores data in the JSONL log. Avoid enabling it in production or on files with long retention.

## Dashboard Health Rules

Open **Health rules**. These settings are stored in the browser, separately from environment variables. They apply to successful dashboard refreshes while the page is open, using the current time range and namespace. Key search and operation filters do not affect the rules.

| Setting | Default | Behaviour |
|---|---|---|
| **Low hit-rate warning** | Enabled | Warns when hit rate is below **Alert below** on the efficiency card. |
| **Alert below** | `50%` | Hit-rate threshold; only cache lookups count towards the sample minimum. |
| **Error-count warning** | Disabled | When enabled, warns at or above the configured number of errors in range. |
| Error-count threshold | `5` | Error count, not a percentage; recorded events count towards the sample minimum. |
| **p95 latency warning** | Disabled | When enabled, warns when p95 cache-operation latency exceeds the threshold. |
| p95 latency threshold | `10 ms` | Only timed operations count towards the sample minimum. |
| **Minimum samples** | `10` | Lookups for hit rate, recorded events for errors, timed operations for latency. |
| **Condition holds for** | `30 seconds` | The condition must remain observed across refreshes for this long before warning. `0` removes the hold period. |
| **Snooze / repeat cooldown** | `60 seconds` | Suppresses snoozed warnings and repeat warnings after recovery. |

Active warnings stay visible until the condition recovers or you select **Snooze**. The cooldown does not hide an active warning automatically. Disable a rule with its checkbox.

Changing settings, the time range, or namespace resets evaluation. A failed refresh also resets it. Reloading starts a new evaluation and clears snoozes, while saved settings remain. There is no background alert service or notification when the dashboard is closed.

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
  "instance": "a1b2c3d4",
  "payload": {
    "key": "users:42",
    "driver": "FileStore",
    "success": true,
    "size_bytes": 2048,
    "duration_ms": 1.9,
    "value_preview": "{\"id\":42,\"name\":\"Alice\"}"
  }
}
```

`value_preview`, `value_type`, `size_bytes`, `count`, and `error` depend on the event and enabled capture. Current v6 records use `hit`, `miss`, `put`, `clear`, `flush`, `prune`, `error`, `promotion`, `stale_served`, `refresh`, and `lock_contended` as event types. Lifecycle markers currently carry no measured duration even if their record contains `duration_ms: 0`; Monitor excludes them from latency statistics.

CacheerPHP 6.1+ events carry namespace, TTL, and operation metadata; 6.0 events do not. Older or custom payloads may also include `namespace` and `ttl`. An explicit empty namespace is the default scope; a missing field is unreported. For writes, explicit `ttl: null` means forever, while a missing `ttl` means unknown. The dashboard explains missing metadata instead of inventing values.

## Live value previews

Captured events are the primary source of previews. When value capture is enabled,
the inspector can also read a live value from FileStore, DatabaseStore, or
RedisStore using the application's shared connection settings (`DB_*`, `REDIS_*`,
and the cache path/table above). These reads do not emit telemetry events.

Live reads use the default storage pipeline and key encoder. For custom codecs,
compression, encryption, or process-local stores such as ArrayStore, enable value
capture in the application. The inspector explains unavailable lookups and keeps
the last captured preview when a live refresh fails. Scoped v6 keys are resolved
only when their display name identifies one stored value; ambiguous names fall
back to captured previews.

## Buffered reporting and rotation

`JsonlReporter` buffers encoded JSONL records and appends each batch under the
events file's shared `.lock`. Each reporter instance attaches an `instance` ID
for multi-process diagnostics. The file rotates at **10 MiB** by default.

A batch flushes at **256 events**, on the first event at least **one second**
after the previous flush, on explicit `$reporter->flush()`, and at process exit,
including PHP fatal errors. PHP-FPM request batches appear when the request
finishes. An idle long-running worker flushes its last batch on its next cache
operation or at exit; call `flush()` at job/request boundaries to write sooner.
A process killed without shutdown, such as `SIGKILL`, can lose its buffered batch.

Processes interleave by batch, so snapshot feeds and key history order records by
timestamp. The snapshot retains the newest matching events before its feed limit
is applied; file order breaks timestamp ties.

A local buffered-reporter benchmark measured about **2.5 microseconds per
`get()` with Monitor installed**. This is workload- and machine-dependent, not a
latency guarantee.

## Local access and response headers

The server binds to `127.0.0.1` by default. The optional token protects destructive
actions; it does not authenticate read endpoints or make public deployment safe.
Add separate access control before exposing the dashboard to a network.

Ordinary API responses use `Cache-Control: no-store`; the SSE stream uses
`no-cache`. Cross-origin API access is off by default. Set
`CACHEER_MONITOR_ALLOW_ORIGIN` only when another origin intentionally needs access;
CORS does not authenticate users. Preview redaction is best effort, so review
captured content before sharing logs or exports.

## Operation and event types by core version

The base mappings are `Hit` → `hit`, `Miss` → `miss`, `Write` → `put`,
`Delete` → `clear`, `Clear` → `flush`, `Prune` → `prune`, and `Failure` → `error`.
Deleting a key and flushing a store are separate operations. Lifecycle types are
`promotion`, `stale_served`, `refresh`, and `lock_contended`.

With CacheerPHP 6.1+, a reported operation distinguishes `touch` → `renew`,
`tag` → `tag`, `increment` → `increment`, and `clearTag` → `flush_tag`.
Loader runs use `computed`; failover uses `fallback_served`, `circuit_opened`,
`circuit_closed`, and `reconciled`. These reports are excluded from cache-operation
latency statistics. Without operation metadata, the base mapping applies.
