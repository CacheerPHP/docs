# API Reference

REST endpoints power the SPA and provide a friendly surface for tooling or scripts.

## REST Endpoints

### `GET /api/health`

Simple liveness probe. Returns `{ "ok": true }` when the server is reachable.

---

### `GET /api/config`

Reports the events file path, how it was resolved (`env`, `.env`, `default`), and other runtime hints.

```json
{
  "events_file": "/tmp/cacheer-monitor.jsonl",
  "origin": "env"
}
```

---

### `GET /api/metrics`

Summarises all events into counters, hit-rate, and latency percentiles.

| Query param | Description |
|---|---|
| `namespace` | Optional. Filter metrics to a single namespace. |

Default limit: 500 recent entries.

```json
{
  "hits": 1200,
  "misses": 45,
  "hit_rate": 0.963,
  "latency": { "avg_ms": 3.1, "p95_ms": 7.8, "p99_ms": 15.2 },
  "drivers": { "redis": 1090, "array": 155 },
  "namespaces": { "default": 800, "critical": 290 },
  "top_keys": { "users:42": 120, "orders:recent": 56 }
}
```

---

### `GET /api/events`

Returns the newest events first. Filter by namespace, type, or match portions of the cache key.

| Query param | Description |
|---|---|
| `limit` | Default `200`. |
| `namespace` | Optional. |

```json
{
  "type": "put",
  "ts": 1714310556,
  "payload": {
    "key": "users:42",
    "namespace": "default",
    "driver": "redis",
    "ttl": 300,
    "duration_ms": 5.4
  }
}
```

---

### `POST /api/events/clear`

> **Destructive.** Use only in local/dev environments.

Rotates the events file: current file is archived with a timestamp suffix and a fresh empty file is created. The dashboard **Clear** button uses this endpoint.

Returns `{ "ok": true, "rotated_to": "..." }`.

---

## Server-Sent Events

### `GET /api/events/stream`

Provides a lightweight heartbeat used by the SPA to refresh the event list as new lines land in the JSONL file. Messages are simple `data: ping` payloads.

When EventSource is not available, the dashboard falls back to interval polling using the configured refresh rate.
