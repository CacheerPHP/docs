# Stale-while-revalidate (`flexible`)

`flexible()` keeps hot data fast **and** fresh. Instead of a hard expiry that
forces a slow recompute at the worst moment, it serves slightly-stale data
instantly while refreshing it in the background.

## The three windows

```php
$feed = $cache->flexible('feed', fresh: 30, stale: 300, callback: fn () => build_feed());
```

Given an entry created at time `T`:

| Age | Behavior |
|---|---|
| `0 .. fresh` (0–30s) | Serve the cached value directly. |
| `fresh .. stale` (30–300s) | Serve the **stale** value immediately, and trigger **one** background refresh. |
| `> stale` (>300s) | The entry is gone; recompute synchronously (single-flight). |

Requirements: `0 < fresh < stale`. The value is stored with a hard TTL of `stale`,
so "stale" data is never older than the `stale` window.

## Why it helps

- **No latency cliff.** Users in the stale window get an instant response; only
  the background refresh pays the recompute cost.
- **No stampede.** The refresh is coordinated by a lock, so exactly one worker
  refreshes even under load.

## Background vs. inline refresh

The refresh runs through the injected **deferred executor**:

- `SyncDeferredExecutor` (default) runs the refresh **inline**, right after the
  stale value is returned — simple, but the current request pays for it.
- `AfterResponseDeferredExecutor` queues the refresh and flushes it after the
  response is sent (on shutdown / `fastcgi_finish_request`), so the user waits for
  neither the stale read nor the refresh.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Support\AfterResponseDeferredExecutor;

$cache = new Cache($store, executor: new AfterResponseDeferredExecutor());
```

CacheerPHP never calls a refresh "background" unless a deferred executor that
actually defers is active. With the sync executor, the refresh is documented — and
behaves — as inline.

## `flexible()` vs. `remember()`

- Use **`remember()`** when a slightly slower response on expiry is acceptable and
  you want the simplest correct behavior.
- Use **`flexible()`** for hot, expensive values where you never want a user to
  wait for a recompute — at the cost of occasionally serving data up to `stale`
  seconds old.

See also the [Policies guide](./policies.md) for `serveStaleOnError`, which serves
stale data specifically when the refresh *fails*.
