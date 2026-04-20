# Cacheer Monitor

Cacheer Monitor is a local dashboard that watches a JSONL events log emitted by CacheerPHP clients. It visualises cache traffic, latency, drivers, and key usage in near-real time so you can debug workloads or showcase Cacheer behaviour.

## What It Does

- Live metrics — hits, misses, puts, flushes, latency percentiles.
- Event stream explorer with filtering by namespace, type, and key.
- Drivers distribution chart, namespace breakdown, top key leaderboard.
- TTL distribution across write events (forever, > 1 day, > 1 hour, etc.).
- Key Inspector — per-key hit/miss history, last TTL, value size, and live value preview.
- Event export — download history as JSON or CSV for offline analysis.
- Value capture — optional recording of cached values in event payloads (redacts sensitive fields automatically).
- Token-protected destructive actions for shared dev environments.
- Lightweight JSONL ingestion with no external dependencies.

## How It Works

1. Cacheer adapters append events to a JSONL file on disk.
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
