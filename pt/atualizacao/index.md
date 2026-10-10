# Atualizando para o CacheerPHP 6

O CacheerPHP 6 é uma reescrita baseada em instâncias. A migração é quase toda
mecânica: renomeie os métodos v5 para os nomes v6 (um conjunto Rector automatiza os
comuns), mova o namespace posicional para `scope()`, e comece a v6 com um cache frio no
seu próprio keyspace. Não há shim de runtime da v5 — a
migração é a renomeação. Se um serviço ainda não puder migrar, mantenha-o em `^5.2`.

## 1. Instalação

```sh
composer require silviooosilva/cacheer-php:"^6.0"
```

A restrição `^6.0` seleciona versões estáveis da linha 6.x e mantém as atualizações
dentro da mesma versão principal.

Consulte o [suporte de versões](./suporte-de-versoes.md) para conhecer as linhas de versões e a política de manutenção.


A v6 exige PHP 8.3+. O núcleo instala sem clientes de backend; `ArrayStore` e
`FileStore` funcionam imediatamente. Redis e drivers PDO continuam opcionais.

## 2. Construção: a seleção de driver vira um construtor nomeado

| v5 | v6 |
|---|---|
| `(new Cacheer())->setDriver()->useFileDriver()` | `Cacheer::file('/var/cache')` |
| `->useDatabaseDriver()` | `Cacheer::database($pdo, 'cacheer')` |
| `->useRedisDriver()` | `Cacheer::redis($connection)` |
| driver de array / testes | `Cacheer::inMemory()` |

O schema do banco **nunca** é criado implicitamente — execute
`DatabaseStoreSchema::migrate($pdo, $table)` (ou `cacheer migrate`) uma vez.

## 3. Mapeamento de métodos

Todo verbo da v5 abaixo é um método do cache na v6 — você nunca precisa alcançar a
store por trás dele, e o escopo em que você está é aplicado para você.

| v5 | v6 | Observações |
|---|---|---|
| `putCache($k, $v, $ns, $ttl)` | `set($k, $v, $ttl)` | Namespace vira `scope($ns)->set(...)` |
| `getCache($k, $ns, $ttl)` | `get($k)` | TTL de leitura removido |
| `clearCache($k, $ns)` / `forget()` | `delete($k)` | `scope($ns)->delete(...)` |
| `flushCache()` | `clear()` | Limitado ao keyspace configurado |
| `forever($k, $v)` | `forever($k, $v)` | Ou `set($k, $v, null)` |
| `add($k, $v, $ns, $ttl)` | `add($k, $v, $ttl)` | Serializado por lock onde a store sabe travar |
| `getAndForget()` / `pull()` | `pull($k, $default = null)` | Lê e remove em uma chamada |
| `has()` | `has()` | — |
| `missing()` | `missing()` | — |
| `getMany()` / `putMany()` | `many()` / `setMany()` | — |
| namespace posicional | `scope('name')` ou `in('name')` | Retorna um cache do mesmo tipo |
| `tag($tag, ...$keys)` | `tag($key, ...$tags)` | Por chave; tags têm namespace por escopo |
| `flushTag($tag)` | `flushTag($tag)` | Retorna quantas foram removidas |
| `increment()` / `decrement()` | `increment()` / `decrement()` | Ambos mantidos |
| `renewCache($k, $ttl, $ns)` | `touch($k, $ttl)` | Estende o TTL, preserva o valor |
| `getAll($ns)` | `entries()` | Escopo aplicado; entrega entradas com metadados |
| `lock($name, $ttl)` | `lock($name, $ttl)` | Nomes de lock têm namespace por escopo |
| `rememberForever()` | `rememberForever()` | — |
| `remember()` / `flexible()` | `remember()` / `flexible()` | Mesma intenção, clock injetado |
| `stats()` | `stats()` | Store, escopo, policy, capacidades reais |
| `useFormatter()` | `formatted()` | Uma visão imutável; leituras base seguem cruas |
| `appendCache()` | ler → mesclar → `set()` | Explícito; use `lock()` se houver concorrência |
| `isSuccess()` / `getMessage()` | `entry()->isHit()` ou retorno | Removido do estado do núcleo |
| `Cacheer::putCache(...)` estático | injete uma instância de `Cache` | Sem estado global na v6 |

As linhas baseadas em capacidade (`increment`, `touch`, `tag`, `flushTag`, `lock`,
`entries`, `prune`) lançam `UnsupportedCapabilityException` em uma store que não as
honra. Todas as stores nativas honram todas; se você suporta backends plugáveis,
pergunte antes com `$cache->supports(AtomicStore::class)`.

### Renomeações automáticas (Rector)

Um conjunto Rector opcional acompanha o pacote em `rector.php`. Ele renomeia os
métodos v5 diretos em `Cacheer` (`putCache`→`set`, `getCache`→`get`,
`renewCache`→`touch`, `getAndForget`→`pull`, …). Os verbos que a v6 manteve — `add`,
`forever`, `missing`, `increment`, `decrement`, `tag`, `flushTag`, `lock`,
`rememberForever`, `stats` — não precisam de regra alguma.

Ele **não** reescreve a construção, não move o argumento de namespace para `scope()`
nem remove o TTL de leitura — faça isso manualmente.

```sh
composer require rector/rector --dev
vendor/bin/rector process src --config vendor/silviooosilva/cacheer-php/rector.php --dry-run
```

## 4. Migrando de forma incremental

Não há shim de runtime da v5, mas você não precisa converter tudo de uma vez:

- Migre um ponto de chamada (ou módulo) por vez para a API `Cacheer`; o mapeamento
  acima e o Rector cobrem a maior parte.
- Mantenha a **mesma store** entre código velho e novo durante a transição — um
  `Cacheer::file(...)` lê o que já está em disco (veja a §5), então código migrado e
  não migrado compartilham dados.
- Se um serviço inteiro ainda não puder mudar, fixe-o em `^5.2` e migre depois. v5 e
  v6 são linhas major diferentes, não duas APIs numa instalação.

## 5. Dados em cache: a v6 começa fria

A v6 não lê os dados em cache da v5. Os layouts de armazenamento e os formatos de
payload são diferentes, então uma store v6 nunca enxerga entradas v5: o primeiro
pedido v6 de cada chave é um miss, e o valor é recalculado e gravado no formato v6.
Cache é dado derivado, então nada se perde — espere um período de aquecimento com
mais misses, como após um flush.

| Backend | Layout v5 | Layout v6 |
|---|---|---|
| Arquivo | `{dir}/[md5(namespace)/]md5(chave).cache` | `{dir}/entries/…` e `{dir}/locks/…` |
| Banco | tabela `cacheer_table` (padrão) | tabela `cacheer_store` (padrão), schema novo |
| Redis | `{namespace}[ns:]chave`, tags como `tag:…` | `{prefix}:e:…`, `{prefix}:t:…`, `{prefix}:l:…` |

Mantenha os dois keyspaces separados:

- **Arquivo**: aponte a v6 para um diretório novo (compartilhar funciona — a v6 só
  mexe em `entries/` e `locks/` — mas um separado torna a limpeza trivial).
- **Banco**: use um nome de tabela diferente da tabela v5.
- **Redis**: use um `$prefix` v6 diferente do namespace v5, ou outro banco lógico.

Depois da janela de rollback, apague os dados v5: o diretório de cache v5, a tabela
v5 ou as chaves Redis v5 (pelo namespace com `SCAN`/`UNLINK`, ou deixando os TTLs
expirarem).

## 6. Migração e rollback do banco

```php
use Silviooosilva\CacheerPhp\Stores\Support\DatabaseStoreSchema;

DatabaseStoreSchema::migrate($pdo, 'cacheer'); // idempotente
DatabaseStoreSchema::drop($pdo, 'cacheer');    // rollback = drop (cache é dado derivado)
```

Veja o DDL sem executar: `cacheer migrate --dry-run`.

## 7. Verificação

- `composer test`, `composer lint`, `composer analyse`
- Rode novamente seus testes de feature (integrações de framework, Redis)

## 8. Plano de rollback

A v6 nunca altera o keyspace da v5. Para voltar: fixe `^5.2` novamente, mantenha o
lock file/vendor anterior e aponte a v5 para o keyspace original. A v6 também nunca
atualizou nem invalidou entradas v5, então **limpe o cache v5 ao fazer rollback**,
a menos que valores cacheados antes da atualização sejam aceitáveis. Depois remova o
keyspace v6 (`cacheer clear --force`).

## Janela de suporte

- **v6** é a linha estável atual e recebe funcionalidades e correções.
- **v5** recebe apenas correções de segurança e de correção por 12 meses após o
  lançamento estável da 6.0. Não há novas funcionalidades para a v5.

> Atualizando da **v4**? Siga primeiro o [guia de migração v5](./v5-migration.md) e
> depois este.
