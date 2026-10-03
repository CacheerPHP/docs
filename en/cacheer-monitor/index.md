# Cacheer Monitor

Cacheer Monitor is a local dashboard that watches a JSONL events log emitted by CacheerPHP clients. It visualises cache traffic, latency, drivers, and key usage in near-real time so you can debug workloads or showcase Cacheer behaviour.

## What It Does

- Live metrics — hits, misses, puts, flushes, latency percentiles.
- Event stream explorer with filtering by namespace, type, and key.
- Drivers distribution chart, namespace breakdown, top key leaderboard.
- Key Inspector — per-key hit/miss history, value size, and live value preview.
- Event export — download history as JSON or CSV for offline analysis.
- Value capture — optional recording of cached values in event payloads (redacts sensitive fields automatically).
- Token-protected destructive actions for shared dev environments.
- Lightweight JSONL ingestion with no external dependencies.

## How It Works

1. The v6 telemetry listener appends typed cache events to a JSONL file on disk.
2. The monitor exposes a small HTTP API backed by that file.
3. The SPA polls (and optionally streams via SSE) metrics + events.
4. CLI helpers manage the dev server and example scenarios.

## Sections

- [Quick Start](quick-start.md)
- [API Reference](api.md)
- [CLI Reference](cli.md)
- [Configuration](configuration.md)
- [Common Workflows](workflows.md)
- [Troubleshooting](troubleshooting.md)

The dashboard can display TTL and namespace breakdowns when event payloads include those fields. The current v6 bridge does not emit TTL or scope metadata; those panels are not a complete view of v6 expiration or scoped keyspaces.
