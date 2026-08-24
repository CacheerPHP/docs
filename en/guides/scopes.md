# Scopes

A **scope** is an isolated keyspace inside one store. Scopes replace v5's
positional string namespaces with a typed, nestable [`Scope`](../api/option-builder.md#scope)
you can clear on its own.

## Basics

```php
$reports = $cache->scope('reports');
$reports = $cache->in('reports');    // in() is an alias — pick whichever reads better

$reports->set('daily', $rows);
$reports->get('daily');          // the rows
$cache->get('daily');            // null — the root scope is a different keyspace
```

The same key name lives independently in each scope:

```php
$cache->scope('reports')->set('daily', $reportRows);
$cache->scope('billing')->set('daily', $invoice);

$cache->scope('reports')->get('daily'); // $reportRows
$cache->scope('billing')->get('daily'); // $invoice
```

## Clearing one scope

`clear()` on a scoped cache removes only that scope — never the whole store:

```php
$cache->scope('reports')->clear();   // only "reports" entries
$cache->clear();                     // the entire store (root)
```

This requires the store to implement `FlushableScopeStore` (all four built-in
stores do). On a store without it, a scoped `clear()` throws
`UnsupportedCapabilityException` rather than clearing too much.

## Nesting

Scopes nest to any depth (up to 64 segments):

```php
$tenant = $cache->scope('tenant:42');
$tenant->scope('reports')->set('daily', $rows);

// Clearing the parent clears its children too.
$tenant->clear();
```

## Validation

Scope segments are validated: they cannot be empty, exceed 255 bytes, or contain
slashes or control characters (`InvalidScopeException`). This keeps scopes safe to
encode into any backend keyspace.

## Scopes vs. tags

- **Scopes** are structural: an entry belongs to exactly one scope, and you clear
  a whole scope at once. Use them to separate features, tenants, or environments.
- **[Tags](../tutorials/example-10-tagging.md)** are cross-cutting labels: an
  entry can carry many tags, and you invalidate by tag. Use them for
  relationships that cut across scopes (e.g. everything touching `user:42`).

## Scope applies to everything, not just get/set

Counters, tags, and locks are scoped too, so two scopes cannot collide:

```php
$cache->in('tenant-a')->increment('signups', 1, initial: 0);
$cache->in('tenant-b')->increment('signups', 5, initial: 0);
// → 1 and 5, in separate keyspaces

$cache->in('tenant-a')->tag('p1', 'products');
$cache->in('tenant-b')->tag('p1', 'products');
$cache->in('tenant-a')->flushTag('products');   // leaves tenant-b alone

$a = $cache->in('tenant-a')->lock('nightly-import', 300);
$b = $cache->in('tenant-b')->lock('nightly-import', 300);
// both acquire — same name, different scopes
```

## Everything is a scope internally

The root cache is just the root scope, and scoping returns the **same type**:

```php
$cache->scope('reports') instanceof Cacheer;   // true
```

So a scoped cache can never silently lack part of the API — it has `remember()`,
`flexible()`, batch reads, capabilities, `withPolicy()`, and `formatted()` just
like the root cache. Ask which scope you are in with `boundScope()`:

```php
$cache->in('tenant:42')->in('reports')->boundScope();   // "tenant:42/reports"
```
