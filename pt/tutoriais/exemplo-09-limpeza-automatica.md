# Removendo entradas expiradas

> A v5 fazia auto-flush em um cronograma como efeito colateral. A v6 torna a limpeza
> explícita: entradas expiradas são removidas preguiçosamente na leitura, e em massa com
> `prune()`.

Stores que implementam `PrunableStore` (as quatro nativas) varrem entradas expiradas e
reportam quantas foram removidas.

```php
use Silviooosilva\CacheerPhp\Cacheer;

$cache = Cacheer::file(__DIR__ . '/cache');

// ... o tempo passa, entradas expiram ...

$removed = $cache->prune();  // número de entradas expiradas apagadas
```

Para ver antes o que ainda está vivo, percorra o keyspace — `entries()` tem o escopo
do cache em que você o chama:

```php
foreach ($cache->in('reports')->entries() as $entry) {
    echo $entry->key()->value(), ' → ', $entry->remainingTtl($clock), "\n";
}
```

Rode pela CLI num cron em vez de no caminho da requisição:

```sh
vendor/bin/cacheer prune --dry-run   # reporta o que seria removido
vendor/bin/cacheer prune             # de fato remove
```

O prune só remove entradas **expiradas** e nunca toca em dados fora do keyspace
configurado. Veja o [guia da CLI](../guias/cli.md).
