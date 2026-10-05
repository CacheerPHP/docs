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
| Value preview missing in Key Inspector | Enable value capture: set `CACHEER_MONITOR_CAPTURE_VALUES=true` in your `.env`. Use the inspector refresh icon (**Refresh live value**) to request a live lookup. The monitor process must be able to resolve the store. |
| `401 Unauthorized` on clear / cleanup | `CACHEER_MONITOR_TOKEN` is set. Include the token as `X-Monitor-Token: <token>` in the request header. |
| Health warning not appearing | Open **Health rules** and enable the relevant checkbox. Check its threshold, **Minimum samples**, and **Condition holds for**. Defaults require 10 samples and an observed breach for 30 seconds; error/latency rules are disabled initially. Snoozed or recovered conditions also respect the cooldown. Empty lookup activity shows `—`, not a hit-rate failure. |
| Warning disappears after reload or changing filters | Saved settings remain, but changing settings, time range, or namespace, reloading, or a failed refresh starts a new evaluation. Snoozes do not survive reload. Key/type filters do not change the rules. |
| Namespace filter is unavailable | Current v6 events do not report namespace metadata. The dashboard marks it unavailable instead of treating all activity as the default scope. Older/custom records can provide an explicit namespace. An active filter stays editable even if no records match. |
| Problem keys is empty | The selected ranking only includes keys with recorded misses, errors, hits, or timed operations. Clear key search, widen the range, or select **Most hits**. No recorded errors is a valid empty result. |
| Lifecycle counts are zero | These count emitted signals, not every lookup. Stale serving, refreshes, tier promotion, and single-flight lock timeouts only appear when those behaviours occur and are instrumented. |
| Feed limit does not change overview metrics | This is expected. Metrics and charts use all matching records in the current log; the limit only changes events shown. Key/type filters also leave overview metrics unchanged. |
| Old history disappears after log rotation | The dashboard reads only the current log, including for **All time** and key inspection. Export or retain archived files separately when you need longer history. |
| Stream says it is reconnecting | SSE connections close after `CACHEER_MONITOR_STREAM_TIMEOUT` (30 seconds by default) and reconnect automatically. **Connected** reports snapshot/API connectivity separately. Interval polling continues at the chosen refresh rate; **Manual** stops polling but allows SSE updates. |
| Time-range filter returns no events | The `from`/`until` params are Unix timestamps in seconds (float). If your events were captured with a different clock offset, widen the range or reset to **All** to confirm events exist. |
