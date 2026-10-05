# Cacheer Monitor

Cacheer Monitor 2.x is a local dashboard for CacheerPHP 6 telemetry. It shows recorded cache traffic, operation latency, lifecycle signals, and problem keys so you can investigate cache behaviour as your application runs.

## What It Does

- Live metrics — hits, misses, puts, flushes, latency percentiles.
- Event stream explorer with filtering by namespace, type, and key.
- Cache lifecycle — stale values served, completed refreshes, tier promotions, and lock-wait timeouts. Select a signal to inspect matching events.
- Problem keys — top 10 matching keys by misses, errors, hits, or p95 latency, grouped by driver and reported namespace.
- Health rules — browser-saved hit-rate, error-count, and latency thresholds, with minimum samples, a hold period, and snooze.
- Drivers distribution chart and namespace/TTL breakdowns when the metadata is reported.
- Key Inspector — driver-specific history, lifecycle counts, value metadata, and an optional live value preview.
- Event export — download history as JSON or CSV for offline analysis.
- Value capture — optional recording of cached values in event payloads (redacts sensitive fields automatically).
- Token-protected destructive actions for shared dev environments.
- Lightweight JSONL ingestion with no external dependencies.

## How It Works

1. The v6 telemetry listener appends typed cache events to a JSONL file on disk.
2. The monitor exposes a small HTTP API backed by that file.
3. The dashboard fetches a snapshot containing metrics, a limited event feed, coverage counts, and chart data. SSE requests a refresh when new events arrive and reconnects after timeouts.
4. CLI helpers manage the dev server and example scenarios.

## Sections

- [Quick Start](quick-start.md)
- [API Reference](api.md)
- [CLI Reference](cli.md)
- [Configuration](configuration.md)
- [Common Workflows](workflows.md)
- [Troubleshooting](troubleshooting.md)

## What the Metrics Cover

Overview metrics use all events in the **current log** that match the selected time range and namespace. The feed limit only changes how many events are shown. Key search filters the feed and key rankings; the operation filter affects the feed. Neither changes overview metrics or health rules.

“All time” means the records retained in the current file. Rotated logs are not included. Charts use 20 buckets across the selected range; with **All time**, they show the last 10 minutes. Rolling windows advance on refresh.

Latency measures recorded cache operations. Untimed lifecycle markers are excluded. The dashboard shows unknown latency when no timed samples exist.

The current v6 bridge does not emit namespace or TTL metadata. Monitor explains when these fields are unavailable; a missing namespace is not assumed to be the default scope, and a missing TTL is not assumed to mean forever. Older or custom records can still supply these fields.

See [Common Workflows](workflows.md) for practical diagnostics and [Configuration](configuration.md) for health-rule defaults.
