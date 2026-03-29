# Quick Start

Spin up the local dashboard in under a minute.

## Install & Run

```bash
composer install
php bin/cacheer-monitor serve --host=127.0.0.1 --port=9966
```

Optionally point to a custom events file before starting:

```bash
export CACHEER_MONITOR_EVENTS=/path/to/cacheer.jsonl
```

Then visit `http://127.0.0.1:9966` in your browser.

## Populate Sample Data

Use the bundled scenarios to emit events covering every driver operation:

```bash
php Tests/scenarios.php
php Tests/scenarios_advanced.php   # bursty load + error cases
```

Each run appends to the configured JSONL file. Use the dashboard **Clear** button or `POST /api/events/clear` to rotate the file.
