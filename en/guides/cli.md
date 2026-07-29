# Operations CLI (`cacheer`)

CacheerPHP 6 ships a small operations CLI at `vendor/bin/cacheer` for health
checks, inspection, and maintenance. Every command supports `--json` for
machine-readable output; mutating commands support `--dry-run`.

## Configuration

Commands that touch a store load an explicit `cacheer.config.php` from the working
directory (or `--config=path`). It returns a `Store`, or an array with a store and
optional PDO/table for `migrate`:

```php
// cacheer.config.php
use Silviooosilva\CacheerPhp\Stores\FileStore;

return new FileStore(__DIR__ . '/var/cache');

// or, for the database:
return [
    'store' => new DatabaseStore($pdo, 'cacheer_store'),
    'pdo'   => $pdo,
    'table' => 'cacheer_store',
];
```

Commands that don't need a store (like `doctor`) run without any config.

## Commands

```sh
vendor/bin/cacheer doctor          # environment & config health check
vendor/bin/cacheer stats           # store name, capabilities, entry count
vendor/bin/cacheer inspect <key>   # metadata for one key — never its value
vendor/bin/cacheer prune           # remove expired entries (PrunableStore)
vendor/bin/cacheer clear --force   # empty the configured keyspace
vendor/bin/cacheer migrate         # create the database schema
vendor/bin/cacheer list            # show available commands
```

### `doctor`

Reports PHP version, loaded extensions, and whether a config was found. Safe to run
anywhere; exits non-zero if something essential is missing.

### `stats`

Names the store, lists the capabilities it implements, and (when inspectable)
counts live entries.

```sh
vendor/bin/cacheer stats --json
# {"store":"FileStore","capabilities":["batch","tags","atomic","locking",...],"entries":128}
```

### `inspect <key>`

Prints hit/miss, timestamps, and remaining TTL for one key — **never the value**,
so it's safe to run against production.

### `prune` / `clear`

Both are keyspace-safe and name their target:

```sh
vendor/bin/cacheer prune --dry-run   # report what would be pruned, delete nothing
vendor/bin/cacheer clear             # refused — clear requires --force
vendor/bin/cacheer clear --force     # empties the configured keyspace only
```

### `migrate`

Creates the `DatabaseStore` schema. Preview the exact DDL first:

```sh
vendor/bin/cacheer migrate --dry-run   # print CREATE TABLE ... without executing
vendor/bin/cacheer migrate             # run it
```

## Safety

- Mutations name the keyspace they affect and support `--dry-run`.
- `clear` additionally requires `--force`, so it can't wipe a store by accident.
- No command ever prints cache values.
