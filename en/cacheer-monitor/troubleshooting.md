# Troubleshooting

When the dashboard looks empty, check these first:

| Symptom | What to check |
|---|---|
| No events visible | Verify the events file path via `GET /api/config` and ensure the file exists |
| Permission error | The PHP process must be able to read **and** write the JSONL file |
| Sluggish UI | Rotate the log with `POST /api/events/clear` if the file has grown large |
| Server won't start | Requires PHP 8.1+ with the JSON extension enabled |
| Stale assets | Hard-refresh the browser after updating monitor assets |
