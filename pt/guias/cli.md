# CLI de operações (`cacheer`)

O CacheerPHP 6 traz uma pequena CLI de operações em `vendor/bin/cacheer` para health
checks, inspeção e manutenção. Todo comando suporta `--json` para saída legível por
máquina; comandos que mutam suportam `--dry-run`.

## Configuração

Comandos que tocam uma store carregam um `cacheer.config.php` explícito do diretório
atual (ou `--config=caminho`). Ele retorna uma `Store`, ou um array com store e
PDO/tabela opcionais para o `migrate`:

```php
// cacheer.config.php
use Silviooosilva\CacheerPhp\Stores\FileStore;

return new FileStore(__DIR__ . '/var/cache');

// ou, para o banco:
return [
    'store' => new DatabaseStore($pdo, 'cacheer_store'),
    'pdo'   => $pdo,
    'table' => 'cacheer_store',
];
```

Comandos que não precisam de store (como `doctor`) rodam sem config.

## Comandos

```sh
vendor/bin/cacheer doctor          # health check de ambiente e config
vendor/bin/cacheer stats           # nome da store, capacidades, contagem de entradas
vendor/bin/cacheer inspect <key>   # metadados de uma chave — nunca o valor
vendor/bin/cacheer prune           # remove entradas expiradas (PrunableStore)
vendor/bin/cacheer clear --force   # esvazia o keyspace configurado
vendor/bin/cacheer migrate         # cria o schema do banco
vendor/bin/cacheer list            # mostra os comandos disponíveis
```

### `doctor`

Reporta a versão do PHP, extensões carregadas e se um config foi encontrado. Seguro de
rodar em qualquer lugar; sai com código diferente de zero se algo essencial faltar.

### `stats`

Nomeia a store, lista as capacidades que ela implementa e (quando inspecionável) conta
as entradas vivas.

```sh
vendor/bin/cacheer stats --json
# {"store":"FileStore","capabilities":["batch","tags","atomic","locking",...],"entries":128}
```

### `inspect <key>`

Imprime hit/miss, timestamps e TTL restante de uma chave — **nunca o valor**, então é
seguro rodar em produção.

### `prune` / `clear`

Ambos são seguros para o keyspace e nomeiam o alvo:

```sh
vendor/bin/cacheer prune --dry-run   # reporta o que seria removido, não apaga nada
vendor/bin/cacheer clear             # recusado — clear exige --force
vendor/bin/cacheer clear --force     # esvazia apenas o keyspace configurado
```

### `migrate`

Cria o schema do `DatabaseStore`. Veja o DDL exato antes:

```sh
vendor/bin/cacheer migrate --dry-run   # imprime CREATE TABLE ... sem executar
vendor/bin/cacheer migrate             # executa
```

## Segurança

- Mutações nomeiam o keyspace afetado e suportam `--dry-run`.
- `clear` ainda exige `--force`, então não apaga uma store por acidente.
- Nenhum comando imprime valores de cache.
