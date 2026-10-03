# Troubleshooting

When the dashboard looks empty, check these first:

| Symptom | What to check |
|---|---|
| No events visible | Verify the events file path via `GET /api/config` and ensure the file exists |
| No `.env` found warning on startup | Create a `.env` at the project root: `cp vendor/cacheerphp/monitor/.env.example .env` — the monitor works without it, but events will go to the system temp dir |
| Permission error | The PHP process must be able to read **and** write the JSONL file |
| Sluggish UI | Rotate the log with `POST /api/events/clear` if the file has grown large |
| Server won't start | Requires PHP 8.3 or newer for the v6 integration. Run `vendor/bin/cacheer-monitor doctor` to verify compatibility. |
| Stale assets | Hard-refresh the browser after updating monitor assets |
| Driver always shows as `unknown` | The v6 listener reads the driver from the typed event's `store` field. Run `vendor/bin/cacheer-monitor doctor`, verify the telemetry listener is active, and confirm that the cache operation emits an instrumented event. No driver reflection or `getCacheStore()` accessor is required. |
| TTL not shown in Key Inspector | The current v6 telemetry bridge does not emit TTL metadata. Configure expiration with `set($key, $value, ttl: ...)` or the builder’s `defaultTtl(...)`; a missing TTL in the dashboard does not mean the entry never expires. |
| Value preview missing in Key Inspector | Enable value capture: set `CACHEER_MONITOR_CAPTURE_VALUES=true` in your `.env`. Then use **Refresh Live** in the inspector to force a live read from the cache store. |
| `401 Unauthorized` on clear / cleanup | `CACHEER_MONITOR_TOKEN` is set. Include the token as `X-Monitor-Token: <token>` in the request header. |
| Hit-rate alert banner not appearing | Enter a non-zero value in the **Alert Threshold %** field and ensure the current hit rate is actually below it. The banner only activates when both conditions are met. If events have not been captured yet, hit rate may show as `0` or `100` depending on the filter window. |
| Time-range filter returns no events | The `from`/`until` params are Unix timestamps in seconds (float). If your events were captured with a different clock offset, widen the range or reset to **All** to confirm events exist. |
