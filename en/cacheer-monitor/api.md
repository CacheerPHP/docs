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

Aggregates matching records in the current log into counters, hit rate, latency percentiles, lifecycle counts, and key rankings. The default limit is 1,000 records; use `limit=0` to aggregate all matching records. Rotated logs are excluded.

| Query param | Default | Description |
|---|---|---|
| `namespace` | — | Filter to an explicitly reported namespace. `(default)` matches an explicit empty namespace, not missing metadata. |
| `limit` | `1000` | Maximum matching records to aggregate. `0` means all. |
| `from` | — | Unix timestamp (float). Include only events at or after this time. |
| `until` | — | Unix timestamp (float). Include only events at or before this time. |

Example (selected fields):

```json
{
  "hits": 2,
  "misses": 1,
  "puts": 1,
  "errors": 0,
  "total_events": 7,
  "latency": { "avg_ms": 2, "p95_ms": 3.7, "p99_ms": 3.94 },
  "latency_samples": 4,
  "namespace_samples": 0,
  "ttl_samples": 0,
  "lifecycle": {
    "stale_served": 1,
    "refresh": 1,
    "promotion": 0,
    "lock_contended": 1
  }
}
```

`latency_samples` counts timed cache operations. Untimed lifecycle markers are excluded; when the count is zero, latency values are zero placeholders and the dashboard displays `—`. `namespace_samples` and `ttl_samples` count records with available metadata. A namespace map alone does not establish that scope metadata was reported. Missing TTL does not increment the forever bucket.

`top_keys` remains a legacy hit-count map. The dashboard uses `problem_keys` instead: four arrays named `misses`, `errors`, `hits`, and `latency`, each containing at most 10 rows sorted by descending count or p95 latency. Only rows with observations for that ranking are included. Each row is grouped by key, driver, and namespace metadata.

Example diagnostic row:

```json
{
  "key": "users:42",
  "driver": "FileStore",
  "namespace": null,
  "hits": 2,
  "misses": 1,
  "errors": 0,
  "writes": 1,
  "events": 7,
  "hit_rate": 0.6666666666666666,
  "latency": { "avg_ms": 2, "p95_ms": 3.7, "p99_ms": 3.94 },
  "latency_samples": 4
}
```

Here `namespace: null` means unreported; `namespace: ""` means explicitly unscoped. `hit_rate` is `null` for keys with no lookups. Latency measures cache operations, not loader, database-query, or request execution time.

---

### `GET /api/snapshot`

Returns the dashboard's metrics, event feed, coverage counts, and chart data from one current-log read. Unlike `/api/metrics`, its `limit` applies only to the feed; metrics and charts use all records matching the time range and namespace.

| Query param | Default | Description |
|---|---|---|
| `limit` | `1000` | Maximum feed events. `0` returns all matching feed events. The dashboard starts at `200`. |
| `namespace` | — | Filter the snapshot to an explicitly reported namespace. |
| `from` | — | Unix timestamp (float), inclusive lower bound. |
| `until` | — | Unix timestamp (float), inclusive upper bound. |
| `key_filter` | Empty | Case-insensitive key fragment; filters the feed and key rankings before their limits. |
| `type` | Empty | Exact event type, such as `lock_contended`; filters the feed before its limit. |

Key/type filters do not change overview metrics, lifecycle counts, or timeline data. Key search only changes rankings and the feed; type only changes the feed.

```bash
curl "http://127.0.0.1:9966/api/snapshot?limit=50&key_filter=users&from=1714310000&until=1714310600"
```

| Response field | Contents |
|---|---|
| `metrics` | Aggregate fields described under `/api/metrics`, including lifecycle counts and diagnostic rankings. |
| `events` | Latest matching feed records, in file order (oldest first). The UI reverses them for display. |
| `coverage.source` | `current_log`; rotated files are excluded. |
| `coverage.matching_events` | Number of records used for metrics after namespace/time filtering. |
| `coverage.matching_feed_events` | Number remaining after key/type filtering, before the feed limit. |
| `coverage.shown_events` | Number returned in `events`. |
| `timeline` | `from`, `until`, `interval_seconds`, and 20 `buckets`. Each bucket has `ts`, `hits`, `misses`, `samples`, and `avg_ms`; average latency is `null` without timed samples. |

With explicit time bounds, the timeline uses that range; bucket width is at least one second. With no `from`, it covers the last 10 minutes ending at `until` or the current time. The dashboard advances rolling bounds on refresh; API timestamps are fixed bounds supplied by the caller.

---

### `GET /api/events`

Returns a JSON array of the latest matching records in file order, oldest first. Namespace/time filters are applied before the limit. Use `/api/snapshot` for key-fragment or event-type filtering.

| Query param | Default | Description |
|---|---|---|
| `limit` | `200` | Maximum number of events returned. |
| `namespace` | — | Filter to a single namespace. |
| `from` | — | Unix timestamp (float). Include only events at or after this time. |
| `until` | — | Unix timestamp (float). Include only events at or before this time. |

```json
[
  {
    "type": "put",
    "ts": 1714310557,
    "instance": "a1b2c3d4",
    "payload": {
      "key": "users:42",
      "driver": "FileStore",
      "duration_ms": 4,
      "success": true
    }
  }
]
```

---

### `GET /api/keys/inspect`

Returns a key summary and recent records, newest first. The summary covers matching key history in the current log independently of `limit` and the dashboard's time window. The UI displays the latest 15 events.

| Query param | Default | Description |
|---|---|---|
| `key` | **required** | The exact cache key to inspect. |
| `namespace` | — | Restrict to a specific namespace. |
| `driver` | — | Restrict history to a recorded driver, for example `FileStore`. |
| `namespace_missing` | `false` | When `true`, only include records without a namespace field. Use this for an unreported-namespace diagnostic row. |
| `limit` | `100` | Maximum number of key events returned. |
| `live` | `false` | When `true` and value capture is enabled, forces a live read from the cache store to populate the value preview. |

Example for `key=users:42&driver=FileStore&namespace_missing=1&limit=1` (selected summary fields):

```json
{
  "summary": {
    "key": "users:42",
    "hits": 2,
    "misses": 1,
    "puts": 1,
    "last_ttl": null,
    "last_ttl_known": false,
    "namespace_samples": 0,
    "latency_samples": 4,
    "lifecycle": { "stale_served": 1, "refresh": 1, "promotion": 0, "lock_contended": 1 },
    "capture_values_enabled": false,
    "drivers": { "FileStore": 7 }
  },
  "events": [
    {
      "type": "lock_contended",
      "ts": 1714310562,
      "instance": "a1b2c3d4",
      "payload": { "key": "users:42", "driver": "FileStore", "duration_ms": 0, "success": true }
    }
  ]
}
```

`last_ttl_known` distinguishes unavailable TTL from an explicitly reported forever value (`last_ttl: null` with `last_ttl_known: true`). Current v6 events do not provide TTL or namespace fields. The summary also includes errors, latency statistics, value metadata, and optional preview information.

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

Streams complete new JSONL records as `data:` messages. Heartbeats are named `ping` events with a timestamp, not literal `data: ping` strings:

```text
data: {"type":"refresh","ts":1714310561,"instance":"a1b2c3d4","payload":{"key":"users:42","driver":"FileStore","duration_ms":0,"success":true}}

event: ping
data: {"ts":1714310563}

```

The connection lasts for `CACHEER_MONITOR_STREAM_TIMEOUT` seconds (default `30`). `EventSource` reconnects automatically after timeout or network failure. The dashboard fetches a snapshot on connection and coalesces event bursts into refreshes; heartbeat pings do not trigger one. Rotation/truncation restarts reading from the new file, and incomplete lines wait for completion.

Interval polling remains available at the selected refresh rate. **Manual** stops polling but does not disable SSE updates. Browser health rules are evaluated after successful snapshot refreshes; `/api/health` remains a server liveness probe and does not report rule results.
