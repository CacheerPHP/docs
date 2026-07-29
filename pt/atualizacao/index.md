# Atualizando para o CacheerPHP 6

O CacheerPHP 6 é uma reescrita baseada em instâncias. Você pode atualizar de duas
formas:

1. **Ponte primeiro, depois modernize.** Troque seu objeto v5 pela ponte
   `LegacyCacheer` — que mantém os nomes de métodos da v5 — e migre as chamadas para
   a API `Cache` no seu ritmo.
2. **Reescreva diretamente.** Substitua as chamadas usando o mapeamento abaixo.

## 1. Instalação

```sh
composer require silviooosilva/cacheer-php:^6.0
```

A v6 exige PHP 8.3+. O núcleo instala sem clientes de backend; `ArrayStore` e
`FileStore` funcionam imediatamente. Redis e drivers PDO continuam opcionais.

## 2. Construção: a seleção de driver vira um construtor nomeado

| v5 | v6 |
|---|---|
| `(new Cacheer())->setDriver()->useFileDriver()` | `Cache::file('/var/cache')` |
| `->useDatabaseDriver()` | `Cache::database($pdo, 'cacheer')` |
| `->useRedisDriver()` | `Cache::redis($connection)` |
| driver de array / testes | `Cache::inMemory()` |

O schema do banco **nunca** é criado implicitamente — execute
`DatabaseStoreSchema::migrate($pdo, $table)` (ou `cacheer migrate`) uma vez.

## 3. Mapeamento de métodos

| v5 | v6 | Observações |
|---|---|---|
| `putCache($k, $v, $ns, $ttl)` | `set($k, $v, $ttl)` | Namespace vira `scope($ns)->set(...)` |
| `forever($k, $v)` | `set($k, $v, null)` | TTL `null` = para sempre |
| `getCache($k, $ns, $ttl)` | `get($k)` | TTL de leitura removido |
| `clearCache($k, $ns)` | `delete($k)` | `scope($ns)->delete(...)` |
| `flushCache()` | `clear()` | Limitado ao keyspace configurado |
| `getAndForget()` / `pull()` | `pull()` (ponte) | Atomicidade reportada por capacidade |
| `has()` / `missing()` | `has()` | — |
| namespace posicional | `scope('name')` | Retorna um cache com escopo |
| `tag($tag, ...$keys)` | `TaggableStore::tag()` | Capacidade, não núcleo |
| `increment()` / `decrement()` | `AtomicStore::increment()` | Capacidade, não núcleo |
| `isSuccess()` | `entry()->isHit()` ou retorno | Removido do estado do núcleo |
| `remember()` / `flexible()` | `remember()` / `flexible()` | Mesma intenção, clock injetado |

### Renomeações automáticas (Rector)

Um conjunto Rector opcional acompanha o pacote em `rector.php`. Ele renomeia os
métodos v5 diretos; **não** reescreve a construção, não move o argumento de
namespace para `scope()` nem remove o TTL de leitura — faça isso manualmente.

```sh
composer require rector/rector --dev
vendor/bin/rector process src --config vendor/silviooosilva/cacheer-php/rector.php --dry-run
```

## 4. A ponte de compatibilidade

```php
use Silviooosilva\CacheerPhp\Compat\LegacyCacheer;

$cache = LegacyCacheer::file('/var/cache');   // ou ::inMemory()
$cache->putCache('user:1', $user, 'accounts', 3600);
$user = $cache->getCache('user:1', 'accounts');
```

Ative as depreciações em desenvolvimento para localizar as chamadas — elas ficam
**silenciosas por padrão**:

```php
$cache = LegacyCacheer::file('/var/cache', emitDeprecations: true);
```

A ponte também expõe `forever`, `has`, `missing`, `pull`/`getAndForget`,
`renewCache`, `increment`/`decrement`, `remember`/`rememberForever`, `tag`/
`flushTag`, `appendCache` e `isSuccess`/`getMessage`.

## 5. Compatibilidade de dados e reescrita na leitura

A v6 grava um envelope autenticado e versionado, mas ainda consegue **ler** valores
v5. Construa o pipeline com um `V5PayloadReader` que corresponda à
compressão/criptografia do seu app v5, e habilite a reescrita na leitura para
recodificar valores antigos no envelope v6:

```php
use Silviooosilva\CacheerPhp\Config\PipelineConfig;
use Silviooosilva\CacheerPhp\Storage\Compat\V5PayloadReader;
use Silviooosilva\CacheerPhp\Stores\FileStore;

$pipeline = PipelineConfig::default()->withV5Reader(new V5PayloadReader(compression: true));
$store = new FileStore('/var/cache', $pipeline->codec(), migrateLegacyOnRead: true);
```

- Os payloads v5 usam AES-256-**CBC** não autenticado; uma chave errada aparece
  como falha de `unserialize`, não criptograficamente. Novas gravações usam o
  envelope v6 autenticado.
- `FileStore` e `DatabaseStore` suportam reescrita na leitura; entradas Redis
  migram na próxima gravação.

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

A reescrita na leitura é opcional, então um rollout somente-leitura nunca altera
dados v5. Para voltar: fixe `^5.2` novamente, mantenha o lock file/vendor anterior e
limpe envelopes exclusivos da v6 (`cacheer clear --force`).

## Janela de suporte

- **v6** é a linha em desenvolvimento ativo.
- **v5** recebe apenas correções de segurança e de correção por 12 meses após o
  lançamento estável da 6.0.

> Atualizando da **v4**? Siga primeiro o [guia de migração v5](./v5-migration.md) e
> depois este.
