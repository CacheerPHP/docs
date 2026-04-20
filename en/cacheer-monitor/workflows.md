# Common Workflows

Practical recipes for daily debugging or demos.

## Reset the Playground

1. Run `POST /api/events/clear` (or click **Clear**).
2. Replay any example script or trigger events from your app.
3. Open the dashboard and set the refresh interval to `1s` for live demos.

## Investigate a Slow Key

1. Filter events by key fragment using the **filter key** input.
2. Inspect latency badges in the event timeline.
3. Cross-check namespaces and drivers to spot anomalies.

## Scope the Dashboard to a Time Window

The header provides quick time-range buttons (**Last 5 min**, **Last 15 min**, **Last 1 h**, **Today**, **All**). Selecting one filters both the metrics cards and the event list to that window.

You can also drive filters programmatically via the API `from`/`until` query params (Unix timestamps):

```bash
NOW=$(date +%s)
curl "http://127.0.0.1:9966/api/metrics?from=$((NOW - 900))&until=$NOW"
```

## Deep-Dive a Cache Key with the Key Inspector

The Key Inspector panel shows per-key hit/miss history, last TTL, value size, value type, and (when value capture is enabled) a live preview of the cached value.

1. Click any key name in the **Recent Events** list or **Top Keys** leaderboard.
2. The inspector slide-in shows the key summary and its full event history.
3. To force a live read from the cache store (not just the log), click **Refresh Live** inside the inspector. This requires `CACHEER_MONITOR_CAPTURE_VALUES=true`.

## Export Event History

Download a snapshot of logged events for offline analysis or archiving:

```bash
# Download as JSON
curl "http://127.0.0.1:9966/api/events/export?format=json" -O -J

# Download as CSV
curl "http://127.0.0.1:9966/api/events/export?format=csv" -O -J

# Scoped to a namespace and time window
curl "http://127.0.0.1:9966/api/events/export?format=csv&namespace=critical&from=1714300000&until=1714400000" -O -J
```

The CSV includes columns: `ts`, `type`, `key`, `namespace`, `driver`, `duration_ms`, `success`, `size_bytes`, `ttl`.

## Clean Up Rotated Archives

Each `POST /api/events/clear` archives the current log with a timestamp suffix. Remove old archives automatically:

```bash
curl -X POST http://127.0.0.1:9966/api/events/cleanup-rotated \
  -H "Content-Type: application/json" \
  -d '{"max_age_days": 7}'
```

## Monitor Hit-Rate Health with the Alert Banner

The dashboard includes a configurable hit-rate threshold. When the overall hit rate drops below the threshold, a red alert banner appears at the top of the page.

1. Enter a threshold percentage (e.g., `80`) in the **Alert Threshold %** input in the header.
2. The banner appears automatically whenever the current hit rate is below that value.
3. Set the value to `0` to disable the alert.

This is useful for catching cache regression during load testing or after a deployment — leave the dashboard open and the banner fires as soon as hit rate degrades.

## Compare Namespaces

Use the namespace filter to isolate traffic. Pair with the donut chart and summary cards to quantify volume per namespace. Export screenshots for reports.

## Automate Reporting

Hit the `/api/metrics` endpoint from CI to capture regression budgets, or pipe JSONL snapshots into analytics pipelines for longer-term comparisons.

```bash
# Metrics for the last hour
NOW=$(date +%s)
HOUR_AGO=$((NOW - 3600))
curl "http://127.0.0.1:9966/api/metrics?from=$HOUR_AGO&until=$NOW"
```
