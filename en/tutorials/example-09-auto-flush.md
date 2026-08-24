# Pruning expired entries

> v5 auto-flushed on a schedule as a side effect. v6 makes cleanup explicit:
> expired entries are removed lazily on read, and in bulk with `prune()`.

Stores that implement `PrunableStore` (all four built-ins) can sweep expired
entries and report how many were removed.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

// ... time passes, entries expire ...

$removed = $cache->prune();  // number of expired entries deleted
```

To see what is still live first, walk the keyspace — `entries()` is scoped to the
cache you call it on:

```php
foreach ($cache->in('reports')->entries() as $entry) {
    echo $entry->key()->value(), ' → ', $entry->remainingTtl($clock), "\n";
}
```

Run it from the CLI on a cron instead of in your request path:

```sh
vendor/bin/cacheer prune --dry-run   # report what would be pruned
vendor/bin/cacheer prune             # actually prune
```

Pruning only ever removes **expired** entries and never touches data outside the
configured keyspace. See the [CLI guide](../guides/cli.md).
