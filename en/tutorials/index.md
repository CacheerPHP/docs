# Tutorials

Short, task-focused examples for CacheerPHP 6. Each one is self-contained and uses
only the core package. For narrative deep dives, see the [Guides](../guides/configuration.md);
for precise signatures, the [API Reference](../api/index.md).

## Basics

1. [Simple data cache](./example-01-simple-data-cache.md) — `set` / `get` / `has` / `delete`
2. [Custom expiration (TTL)](./example-02-custom-expiration-ttl.md) — every TTL format
3. [Cleaning and flushing](./example-03-cleaning-and-flushing.md) — `delete`, `clear`, scoped clear
4. [Scopes](./example-04-namespaces.md) — isolated keyspaces
5. [Caching an API response](./example-05-api-response-cache.md) — `remember()`
6. [The CacheEntry object](./example-06-json-output-formatter.md) — hit/miss, timestamps, TTL

## Working with data

7. [Batch operations](./example-07-array-output-formatter.md) — `many` / `setMany` / `deleteMany`
8. [Encryption & compression](./example-08-string-output-formatter.md) — the storage pipeline
9. [Pruning expired entries](./example-09-auto-flush.md) — `prune()` and the CLI
10. [Tagging](./example-10-tagging.md) — group and invalidate by tag
11. [PSR-16 adapter](./example-11-psr16-adapter.md) — standard SimpleCache
12. [DateInterval TTL](./example-12-dateinterval-ttl.md) — native PHP intervals
13. [Falsy and null values](./example-13-falsy-values.md) — a cached `null` is a hit

## Advanced

14. [Atomic counters & compare-and-swap](./example-14-add-conditional.md)
15. [Observability: events & metrics](./example-15-stats-instance.md)
16. [Tiered caching (L1/L2)](./example-16-monitor-integration.md)
17. [Locks & critical sections](./example-17-locks-atomic-counters.md)
18. [Stampede protection & stale-while-revalidate](./example-18-stampede-and-swr.md)

See also the [Resilient store](../guides/resilient-store.md) and
[Policies](../guides/policies.md) guides.
