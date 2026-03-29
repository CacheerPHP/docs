# Configuration & Environment

The monitor favours convention, yet a few knobs keep it flexible during development.

## Events File Resolution

The monitor resolves the events file in this order:

1. **OS environment** — `CACHEER_MONITOR_EVENTS`
2. **`.env` file** — read from the project root (see `src/Support/Env.php`)
3. **Fallback** — `sys_get_temp_dir()/cacheer-monitor.jsonl`

Relative paths are resolved against the repository root.

## Refresh Rate

- The UI refresh interval defaults to **5 s** (adjustable via the select in the header).
- Manual refresh is available using the dashboard **Refresh** button.
- The event stream uses SSE ping fallbacks when supported.
- For large files, consider pruning via `POST /api/events/clear` after archiving.

## Event Payload Format

Each line in the JSONL file is a single Cacheer event:

```json
{
  "type": "hit",
  "ts": 1714310556,
  "payload": {
    "key": "users:42",
    "namespace": "default",
    "driver": "redis",
    "size_bytes": 2048,
    "duration_ms": 1.9,
    "tags": ["groupA", "beta-testers"]
  }
}
```

Additional fields (e.g., `ttl`, `exception`) appear depending on the event type.
