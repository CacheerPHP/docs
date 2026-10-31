# Cacheer Monitor

Cacheer Monitor 2.x is a local dashboard for CacheerPHP 6 telemetry. It shows recorded cache traffic, operation latency, lifecycle signals, and problem keys so you can investigate cache behaviour as your application runs.

## Dashboard features

| Feature | Description |
| --- | --- |
| **Metric cards** | Hits, misses, puts, flushes, renews, increments, tag flushes, clears, errors, avg latency (p95/p99) |
| **Cache efficiency** | Hit rate, lookup count, and a horizontal meter; no lookups is shown explicitly |
| **Health rules** | Hit-rate, error-count, and p95 latency warnings; minimum samples, hold period, and snooze cooldown |
| **Hits vs Misses chart** | Selected rolling time range in 20 buckets; All time shows the last 10 minutes |
| **Latency chart** | Avg latency over time |
| **TTL distribution** | Expiry buckets when events report TTL; missing metadata is shown explicitly |
| **Drivers doughnut** | Event count breakdown by driver |
| **Problem keys** | Top 10 matching keys per ranking, grouped by driver and namespace; hits, misses, errors, writes, and latency |
| **Key patterns** | Keys grouped by shape (`{id}`, `{uuid}`, `{ulid}`, `{date}`, `{email}`, `{hash}`) per driver and namespace; rank by lowest hit rate (patterns with 20+ lookups), time saved, wasted writes, misses, lookups, or p95 latency |
| **Wasted work** | Wasted writes (overwritten, deleted, or expired unread) and recent stampedes with their misses, processes, and loader runs |
| **Time saved** | Estimated loader time saved, in the efficiency card and per key pattern, with the loader runs it is based on (CacheerPHP 6.1+) |
| **Cache lifecycle** | Stale served, refreshes, promotions, and lock contention; select a signal to filter events |
| **Failover** | Served by fallback, circuit opened/closed, reconciled, and outage windows newest first (CacheerPHP 6.1+) |
| **Namespaces grid** | Counts for reported namespaces; unavailable metadata is explained |
| **Event stream** | Live feed with type badges, key, driver, duration, TTL, size, value type |
| **Key Inspector** | Driver-specific history, lifecycle counts, timestamps, value preview, and an event timeline |
| **Value preview** | Captured or live-resolved cache values with automatic sensitive field redaction |
| **Export** | Download events as JSON or CSV |
| **SSE real-time** | Server-Sent Events push updates to the dashboard |
| **Auto-refresh** | Configurable: off, 1s, 2s, 5s, 10s — persisted to localStorage |
| **Time-range filter** | 5m, 15m, 1h, 6h, 24h, All |
| **Dark / Light theme** | Toggle with FOUC prevention — persisted to localStorage |
| **Activity indicator** | Subtle progress line while requests are in flight; respects reduced motion |
| **Workspace navigation** | Persistent desktop rail, compact mobile navigation, and active section highlighting |
| **Local assets** | Bundled fonts, SVG icons, and Chart.js; the dashboard needs no CDN access |
| **Keyboard access** | Focus indicators, accessible key buttons, and an inspector with focus trapping and Escape dismissal |

## How It Works

1. The v6 telemetry listener appends typed cache events to a JSONL file on disk.
2. The monitor exposes a small HTTP API backed by that file.
3. The dashboard fetches a snapshot containing metrics, a limited event feed, coverage counts, and chart data. SSE requests a refresh when new events arrive and reconnects after timeouts.
4. CLI helpers start the local server and diagnose the telemetry bridge.

## Sections

- [Quick Start](quick-start.md)
- [API Reference](api.md)
- [CLI Reference](cli.md)
- [Configuration](configuration.md)
- [Common Workflows](workflows.md)
- [Troubleshooting](troubleshooting.md)
- [Contributing](../contributing/cacheer-monitor.md)

## What the Metrics Cover

Overview metrics use all events in the **current log** that match the selected time range and namespace. The feed limit only changes how many events are shown. Key search filters the feed and key rankings; the operation filter affects the feed. Neither changes overview metrics or health rules.

“All time” means the records retained in the current file. Rotated logs are not included. Charts use 20 buckets across the selected range; with **All time**, they show the last 10 minutes. Rolling windows advance on refresh.

Latency measures recorded cache operations, not application requests, loaders, or database queries. Untimed lifecycle markers, loader runs, and failover reports are excluded. Rankings show timed sample counts alongside average and p95 latency; a small sample is not a stable estimate. The dashboard shows unknown latency when no timed samples exist.

CacheerPHP 6.1+ reports namespace (scope), TTL, and operation metadata. CacheerPHP 6.0 does not. Missing namespace is unknown, not the default scope; missing TTL is unknown, not forever. An explicitly reported `ttl: null` means forever. Failover reports and loader timings also require 6.1; see [Common Workflows](workflows.md).

See [Common Workflows](workflows.md) for practical diagnostics and [Configuration](configuration.md) for health-rule defaults.
