# API Reference

REST endpoints power the SPA and provide a friendly surface for tooling or scripts.

## REST Endpoints

### `GET /api/health`

Simple liveness probe. Returns `{ "ok": true }` when the server is reachable.

---

### `GET /api/config`

Reports the events file path, how it was resolved (`env`, `dotenv`, `default`), and other runtime hints.

```json
{
  "events_file": "/tmp/cacheer-monitor.jsonl",
  "origin": "env"
}
```

---

### `GET /api/metrics`

Summarises all events into counters, hit-rate, and latency percentiles.

| Query param | Default | Description |
|---|---|---|
| `namespace` | — | Filter metrics to a single namespace. |
| `limit` | `1000` | Maximum number of events to aggregate. |
| `from` | — | Unix timestamp (float). Include only events at or after this time. |
| `until` | — | Unix timestamp (float). Include only events at or before this time. |

```json
{
  "hits": 1200,
  "misses": 45,
  "hit_rate": 0.963,
  "latency": { "avg_ms": 3.1, "p95_ms": 7.8, "p99_ms": 15.2 },
  "drivers": { "redis": 1090, "array": 155 },
  "namespaces": { "default": 800, "critical": 290 },
  "top_keys": { "users:42": 120, "orders:recent": 56 },
  "ttl_distribution": {
    "forever": 10, "gt_1day": 40, "gt_1hour": 120,
    "gt_5min": 80, "gt_1min": 30, "lte_1min": 5
  }
}
```

---

### `GET /api/events`

Returns the newest events first. Filterable by namespace, type, or key fragment.

| Query param | Default | Description |
|---|---|---|
| `limit` | `200` | Maximum number of events returned. |
| `namespace` | — | Filter to a single namespace. |
| `from` | — | Unix timestamp (float). Include only events at or after this time. |
| `until` | — | Unix timestamp (float). Include only events at or before this time. |

```json
{
  "type": "put",
  "ts": 1714310556,
  "payload": {
    "key": "users:42",
    "namespace": "default",
    "driver": "FileCacheStore",
    "ttl": 86400,
    "duration_ms": 5.4
  }
}
```

---

### `GET /api/keys/inspect`

Returns a per-key summary and the most recent events for that key. Powers the Key Inspector panel in the dashboard.

| Query param | Default | Description |
|---|---|---|
| `key` | **required** | The exact cache key to inspect. |
| `namespace` | — | Restrict to a specific namespace. |
| `limit` | `100` | Maximum number of key events returned. |
| `live` | `false` | When `true` and value capture is enabled, forces a live read from the cache store to populate the value preview. |

```json
{
  "summary": {
    "key": "users:42",
    "hits": 18,
    "misses": 2,
    "puts": 3,
    "hit_rate": 0.9,
    "last_put_at": 1714310556,
    "last_hit_at": 1714310900,
    "last_miss_at": 1714309100,
    "last_ttl": 86400,
    "last_size_bytes": 512,
    "last_value_type": "array",
    "last_value_preview": "{\"id\":42,\"name\":\"Alice\"}",
    "capture_values_enabled": true,
    "namespaces": { "default": 20 },
    "drivers": { "FileCacheStore": 20 }
  },
  "events": [ ... ]
}
```

---

### `GET /api/events/export`

Downloads a snapshot of stored events as JSON or CSV.

| Query param | Default | Description |
|---|---|---|
| `format` | `json` | `json` or `csv`. |
| `limit` | `0` (all) | Maximum number of events to export. `0` means no limit. |
| `namespace` | — | Filter to a single namespace. |
| `from` | — | Unix timestamp (float). |
| `until` | — | Unix timestamp (float). |

The response includes appropriate `Content-Disposition` and `Content-Type` headers for direct download.

CSV columns: `ts`, `type`, `key`, `namespace`, `driver`, `duration_ms`, `success`, `size_bytes`, `ttl`.

---

### `POST /api/events/clear`

> **Destructive.** Use only in local/dev environments.

Rotates the events file: current file is archived with a timestamp suffix and a fresh empty file is created. The dashboard **Clear** button uses this endpoint.

Returns `{ "ok": true }`.

If `CACHEER_MONITOR_TOKEN` is set, this endpoint requires the token via the `X-Monitor-Token` header:

```http
POST /api/events/clear HTTP/1.1
X-Monitor-Token: your-secret-token
```

Omitting or mismatching the token returns `401 Unauthorized`.

---

### `POST /api/events/cleanup-rotated`

Deletes archived (rotated) event files older than a given number of days.

| Body field | Default | Description |
|---|---|---|
| `max_age_days` | `7` | Delete rotated archives older than this many days. Minimum `1`. |

```json
{ "max_age_days": 14 }
```

Returns `{ "ok": true, "deleted": 3 }` where `deleted` is the number of files removed.

---

## Server-Sent Events

### `GET /api/events/stream`

Provides a lightweight heartbeat used by the SPA to refresh the event list as new lines land in the JSONL file. Messages are simple `data: ping` payloads.

The connection is held open for `CACHEER_MONITOR_STREAM_TIMEOUT` seconds (default `30`). When `EventSource` is not available, the dashboard falls back to interval polling using the configured refresh rate.
