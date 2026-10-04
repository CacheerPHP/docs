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
| `> stale` (>300s) | Never served: recompute synchronously (single-flight). |

Requirements: `0 < fresh < stale`. The value is stored with a hard TTL of `stale`,
and age is measured from the value's creation time, so a value older than the
`stale` window is never served — even one written under the same key with a
longer TTL, or promoted from another cache layer.

## Why it helps

- **No latency cliff.** Users in the stale window get an instant response; only
  the background refresh pays the recompute cost.
- **No stampede.** A burst of stale reads queues **one** refresh per key: the
  refresh lock marks it pending until it finishes, fails, or can't be scheduled
  (across processes, for as long as the lock's lease lasts). A queued refresh
  re-checks freshness first, so it does nothing if the value was already
  refreshed. On a store without locking, each stale read queues a task, but only
  the first computes.

## Background vs. inline refresh

The refresh runs through the injected **deferred executor**:

- `SyncDeferredExecutor` (default) runs the refresh **inline**, right after the
  stale value is returned — simple, but the current request pays for it.
- `AfterResponseDeferredExecutor` queues the refresh and flushes it after the
  response is sent (on shutdown / `fastcgi_finish_request`), so the user waits for
  neither the stale read nor the refresh.

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Support\AfterResponseDeferredExecutor;

$cache = new Cacheer($store, executor: new AfterResponseDeferredExecutor());
```

CacheerPHP never calls a refresh "background" unless a deferred executor that
actually defers is active. With the sync executor, the refresh is documented — and
behaves — as inline.

### Long-running workers

`AfterResponseDeferredExecutor` flushes on shutdown. In a worker that serves many
requests or jobs in one process (queue workers, RoadRunner, Swoole, FrankenPHP
worker mode), shutdown only happens at the end, so flush explicitly after each unit
of work:

```php
$executor = new AfterResponseDeferredExecutor();
$cache = new Cacheer($store, executor: $executor);

foreach ($jobs as $job) {
    handle($job, $cache);
    $executor->flush(); // run this job's queued refreshes now
}
```

## `flexible()` vs. `remember()`

- Use **`remember()`** when a slightly slower response on expiry is acceptable and
  you want the simplest correct behavior.
- Use **`flexible()`** for hot, expensive values where you never want a user to
  wait for a recompute — at the cost of occasionally serving data up to `stale`
  seconds old.

See also the [Policies guide](./policies.md) for `serveStaleOnError`, which serves
stale data specifically when the refresh *fails*.
