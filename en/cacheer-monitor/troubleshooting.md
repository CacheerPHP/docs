# Troubleshooting

When the dashboard looks empty, check these first:

| Symptom | What to check |
|---|---|
| No events visible | Verify the events file path via `GET /api/config` and ensure the file exists |
| No `.env` found warning on startup | Create a `.env` at the project root: `cp vendor/silviooosilva/cacheer-php/.env.example .env` — the monitor works without it, but events will go to the system temp dir |
| Permission error | The PHP process must be able to read **and** write the JSONL file |
| Sluggish UI | Rotate the log with `POST /api/events/clear` if the file has grown large |
| Server won't start | Requires PHP 8.1+ with the JSON extension enabled |
| Stale assets | Hard-refresh the browser after updating monitor assets |
| Driver always shows as `unknown` | The monitor resolves the driver name via reflection. If your CacheerPHP version exposes `getCacheStore()`, it will be used; otherwise the monitor falls back to reading the public `$cacheStore` property. If neither is available, `unknown` is returned. Upgrade to a supported version or check that the cache store is initialised before the first instrumented call. |
| TTL not shown in Key Inspector | TTL is captured from the `putCache` / `add` argument list at position 3 (i.e., `putCache($key, $value, $namespace, $ttl)`). If you set TTL via `OptionBuilder` (e.g., `->expirationTime()->day(30)`), the monitor reads it from `$cacheer->options['expirationTime']` automatically. If TTL still doesn't appear, confirm you are using cacheer-monitor `>= 1.0.0` which includes the `extractNsAndTtl` fix. |
| Value preview missing in Key Inspector | Enable value capture: set `CACHEER_MONITOR_CAPTURE_VALUES=true` in your `.env`. Then use **Refresh Live** in the inspector to force a live read from the cache store. |
| `401 Unauthorized` on clear / cleanup | `CACHEER_MONITOR_TOKEN` is set. Include the token as `X-Monitor-Token: <token>` in the request header. |
| Hit-rate alert banner not appearing | Enter a non-zero value in the **Alert Threshold %** field and ensure the current hit rate is actually below it. The banner only activates when both conditions are met. If events have not been captured yet, hit rate may show as `0` or `100` depending on the filter window. |
| Time-range filter returns no events | The `from`/`until` params are Unix timestamps in seconds (float). If your events were captured with a different clock offset, widen the range or reset to **All** to confirm events exist. |
