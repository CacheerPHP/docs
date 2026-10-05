# Common Workflows

Practical recipes for daily debugging or demos.

## Reset the Playground

1. Run `POST /api/events/clear` (or open **Log maintenance** and click **Clear events**).
2. Replay any example script or trigger events from your app.
3. Open the dashboard and set the refresh interval to `1s` for live demos.

## Investigate a Slow Key

1. Open **Problem keys** in **Cache explorer** and choose **Highest p95 latency** under **Rank by**.
2. Compare average/p95 latency and the number of timed samples. One slow sample is not a stable estimate.
3. Use **Search keys…** to narrow the rankings and event feed. Search is applied before the top 10 are selected.
4. Click a key to inspect its history for that driver and reported namespace.

Latency measures cache operations, not request duration or time spent computing a value. Lifecycle markers without duration are excluded.

## Find Keys That Miss or Fail

Choose **Most misses** or **Most errors** in **Problem keys**. Keys with no observations for that ranking are omitted; a key that only misses can still appear. The table also shows hits, hit rate, writes, and timed samples. **Most hits** provides the activity ranking.

Identical key strings in different drivers or reported namespaces remain separate. A namespace marked **unreported** is not the default scope.

## Investigate Cache Lifecycle Signals

The **Cache lifecycle** panel counts recorded signals in the current range:

| Signal | Meaning |
|---|---|
| **Stale served** | An older cached value was returned. |
| **Refreshes** | Refresh work completed. |
| **Promotions** | A tiered store promoted a value to its faster layer. |
| **Lock contention** | A single-flight lock wait timed out. This does not count every lock wait. |

Select a signal to filter the event feed, then click a key to open its inspector. The inspector shows lifecycle counts and an event timeline for that key and driver. Clear a key search if it hides events you expected to see.

## Scope the Dashboard to a Time Window

The header provides **5m**, **15m**, **1h**, **6h**, **24h**, and **All time**. A selected rolling window advances on each refresh and applies to overview metrics, lifecycle counts, rankings, and the event feed. Charts use 20 buckets across that range; **All time** shows the last 10 minutes in the charts.

Metrics include all matching records in the current log regardless of the feed limit. Rotated files are excluded. Key and operation filters narrow the feed without changing overview metrics or health rules; key search also narrows rankings.

You can also drive filters programmatically via the API `from`/`until` query params (Unix timestamps):

```bash
NOW=$(date +%s)
curl "http://127.0.0.1:9966/api/snapshot?limit=50&from=$((NOW - 900))&until=$NOW"
```

## Deep-Dive a Cache Key with the Key Inspector

The Key Inspector shows recorded hits, misses, writes, hit rate, lifecycle counts, value metadata, and the latest 15 events, newest first. Its summary covers matching key history in the current log, independently of the dashboard's time window.

1. Click a key in **Event stream** or **Problem keys**.
2. Review its lifecycle counts and event timeline for the selected driver and namespace metadata.
3. To request a live value lookup, use the inspector refresh icon (**Refresh live value**). This requires `CACHEER_MONITOR_CAPTURE_VALUES=true`; a live preview is only available when the store can be resolved by the monitor process.

The current v6 events do not report TTL or namespace metadata. **Not reported** describes missing metadata, not an entry that never expires.

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

## Configure Dashboard Health Rules

1. Open **Health rules** below **Cache lifecycle**.
2. Keep **Low hit-rate warning** enabled and choose **Alert below** on the efficiency card, or enable **Error-count warning** and **p95 latency warning** with their thresholds.
3. Set **Minimum samples**, **Condition holds for**, and **Snooze / repeat cooldown**. See [Configuration](configuration.md) for defaults and the sample count used by each rule.
4. Leave the dashboard open. Conditions are evaluated on successful refreshes for the current time range and namespace.
5. Use **Snooze** to hide a warning for the cooldown period. Disable a rule with its checkbox.

Settings persist in this browser. Warnings remain visible until recovery or snooze; a recovered condition respects the repeat cooldown before warning again. Changing settings, time range, or namespace, reloading, or losing the connection starts a fresh evaluation. There are no background notifications while the dashboard is closed.

## Compare Namespaces

Use the namespace filter when events explicitly report that field. Pair it with the driver chart and summary cards to compare recorded traffic. The current v6 bridge does not provide namespace metadata, so the filter is marked unavailable when the displayed records lack it. A `(default)` filter matches explicitly unscoped records, not records with missing metadata.

## Automate Reporting

Read `/api/metrics` for a report, or archive JSONL exports for longer-term comparisons. Use `limit=0` to aggregate all matching records in the current log; `/api/metrics` otherwise defaults to 1,000 events.

```bash
# Metrics for the last hour
NOW=$(date +%s)
HOUR_AGO=$((NOW - 3600))
curl "http://127.0.0.1:9966/api/metrics?limit=0&from=$HOUR_AGO&until=$NOW"
```
