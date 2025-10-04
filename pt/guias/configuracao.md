# Configuração

O CacheerPHP carrega configurações de variáveis de ambiente usando [vlucas/phpdotenv](https://github.com/vlucas/phpdotenv). Copie o arquivo `.env.example` para `.env` e ajuste os valores para o seu ambiente.

```sh
cp .env.example .env
```

## Banco de dados

| Variável          | Descrição                                              | Padrão          |
| ----------------- | ------------------------------------------------------ | ----------------|
| `DB_CONNECTION`   | Driver do banco (`mysql`, `pgsql`, `sqlite`)          | `sqlite`        |
| `DB_HOST`         | Host do banco                                         | `localhost`     |
| `DB_PORT`         | Porta do banco                                        | `3306`          |
| `DB_DATABASE`     | Nome do banco                                         | `cacheer_db`    |
| `DB_USERNAME`     | Usuário do banco                                      | `root`          |
| `DB_PASSWORD`     | Senha do banco                                        | (vazio)         |
| `CACHEER_TABLE`   | Tabela para o driver de banco                         | `cacheer_table` |

## Redis

| Variável          | Descrição                                              | Padrão      |
| ----------------- | ------------------------------------------------------ | ------------|
| `REDIS_CLIENT`    | Cliente Redis (ex.: `predis`)                          | (vazio)     |
| `REDIS_HOST`      | Host do Redis                                          | `localhost` |
| `REDIS_PASSWORD`  | Senha do Redis                                         | (vazio)     |
| `REDIS_PORT`      | Porta do Redis                                         | `6379`      |
| `REDIS_NAMESPACE` | Namespace opcional para prefixar chaves               | (vazio)     |

Essas variáveis são lidas na inicialização em `src/Boot/Configs.php`, permitindo ao CacheerPHP conectar-se aos backends preferidos.
