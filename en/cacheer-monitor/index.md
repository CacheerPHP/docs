# Cacheer Monitor

Cacheer Monitor is a local dashboard that watches a JSONL events log emitted by CacheerPHP clients. It visualises cache traffic, latency, drivers, and key usage in near-real time so you can debug workloads or showcase Cacheer behaviour.

## What It Does

- Live metrics — hits, misses, puts, flushes, latency percentiles.
- Event stream explorer with filtering by namespace, type, and key.
- Drivers distribution chart, namespace breakdown, top key leaderboard.
- Lightweight JSONL ingestion with no external dependencies.

## How It Works

1. Cacheer adapters append events to a JSONL file on disk.
2. The monitor exposes a small HTTP API backed by that file.
3. The SPA polls (and optionally streams) metrics + events.
4. CLI helpers manage the dev server and example scenarios.

## Sections

- [Quick Start](quick-start.md)
- [API Reference](api.md)
- [CLI Reference](cli.md)
- [Configuration](configuration.md)
- [Common Workflows](workflows.md)
- [Troubleshooting](troubleshooting.md)
