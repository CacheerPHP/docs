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

## Compare Namespaces

Use the namespace filter to isolate traffic. Pair with the donut chart and summary cards to quantify volume per namespace. Export screenshots for reports.

## Automate Reporting

Hit the `/api/metrics` endpoint from CI to capture regression budgets, or pipe JSONL snapshots into analytics pipelines for longer-term comparisons.
