# Configurar o CacheerPHP para Laravel

Instalar o pacote encaminha as stores que já tem através do CacheerPHP; a primeira secção trata disso. O resto da página trata do driver `cacheer`. Cada store `cacheer` é uma entrada em `config/cache.php`, e pode definir várias, por exemplo uma store Redis como predefinida e uma store de ficheiros para dados locais.

## Encaminhar as stores existentes

O encaminhamento não precisa de configuração. Cada store que o Laravel constrói passa a correr através do CacheerPHP: as leituras e escritas passam por ele no formato da própria store e com as mesmas chaves, enquanto `add()`, contadores, `touch()`, locks e tags vão diretamente para a store, com a semântica e os resultados dela.

Para deixar uma store como está, acrescente-lhe `'cacheer' => false`:

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_CACHE_CONNECTION', 'cache'),
    'cacheer' => false,
],
```

Para desligar o encaminhamento em todas as stores, acrescente uma entrada `cacheer` a `config/cache.php`, ao lado de `stores`, e defina `CACHEER_ROUTE=false`:

```php
'cacheer' => [
    'route' => env('CACHEER_ROUTE', true),
],
```

Algumas stores ficam nativas, porque o próprio código do Laravel depende delas de formas que o encaminhamento não consegue reproduzir:

| Store | Porque não é encaminhada |
| --- | --- |
| Uma store `cacheer` | Já corre sobre o CacheerPHP. |
| `array`, e outras stores sem `add()` ou locks | O repositório de cache do Laravel emula `add()` para elas, e o encaminhamento tem de reproduzir cada store com exatidão. |
| `null` | Não guarda nada. |
| `failover`, e `Cache::memo()` | Envolvem outras stores, que são encaminhadas por si. |
| DynamoDB | Os mutexes do scheduler e dos comandos do Laravel verificam se a store é uma `DynamoDbStore`. |
| Redis com a opção `serializer` ou `compression` do phpredis | O rate limiter do Laravel verifica se a store é uma `RedisStore` para contornar essa serialização. |

`php artisan cacheer:doctor` lista cada store, com o motivo quando não é encaminhada. Se outro pacote substituir o gestor de cache do Laravel, nenhuma store é encaminhada, e o `cacheer:doctor` indica-o.

## Opções

Uma store `cacheer` aceita estas opções.

| Opção | Predefinição | Descrição |
| --- | --- | --- |
| `driver` | — | Sempre `cacheer`. |
| `backend` | `file` | Onde as entradas são guardadas: `file`, `array`, `database` ou `redis`. |
| `path` | `storage/framework/cache/cacheer` | Diretório do backend `file`. |
| `connection` | `default` para redis; a ligação predefinida para database | Uma ligação de `config/database.php`. |
| `table` | `cacheer_store` | Tabela do backend `database`. |
| `redis_prefix` | `cacheer` | Prefixo das chaves no Redis, para o backend `redis`. |
| `prefix` | `cache.prefix` | O espaço de chaves isolado da store. Consulte [prefixo](#prefixo). |
| `fallback` | nenhum | Um backend a usar enquanto o principal está em baixo: o nome de um backend, como `'file'`, ou um array de backend completo. |
| `l1` | nenhum | Manter cópias no processo à frente da store: um número de segundos (a idade máxima) ou `true` para 60. `0` desliga-a. |
| `encryption` | nenhum | `['active' => id, 'passphrases' => [id => passphrase]]`. |
| `compress` | `false` | `true`, ou um nível de gzip de 1 a 9. `0` desliga-a. |
| `max_value_bytes` | nenhum | Rejeitar valores maiores na escrita e na leitura. `0` significa sem limite. |

`l1`, `compress` e `max_value_bytes` também aceitam strings numéricas, tal como `env()` as devolve, e uma string vazia, como uma variável por definir, mantém-nas desligadas. Um valor inválido é recusado quando a store é criada, com o nome da opção no erro.

## Backends

```php
// Ficheiros neste host, sem serviços.
'cacheer' => ['driver' => 'cacheer', 'backend' => 'file'],

// Uma ligação redis do Laravel.
'cacheer' => ['driver' => 'cacheer', 'backend' => 'redis', 'connection' => 'cache'],

// Uma ligação de base de dados do Laravel. Execute primeiro a migration (consulte a instalação).
'cacheer' => ['driver' => 'cacheer', 'backend' => 'database', 'connection' => 'mysql'],

// Memória do processo, para testes.
'cacheer' => ['driver' => 'cacheer', 'backend' => 'array'],
```

O backend redis lê o host, a porta, a palavra-passe e a base de dados da ligação do Laravel que indicar, e abre a sua própria ligação com esses dados. Não usa as opções `prefix`, `serializer` nem `compression` do redis do Laravel: o CacheerPHP aplica prefixos e serializa por conta própria, e um cliente que altere chaves ou valores estragaria as entradas guardadas. Os clientes `phpredis` e `predis` funcionam; clusters Redis não são suportados.

## Failover

Servir a partir de outro backend enquanto o Redis está em baixo:

```php
'cacheer' => [
    'driver' => 'cacheer',
    'backend' => 'redis',
    'connection' => 'cache',
    'fallback' => 'file',
],
```

As leituras e escritas passam para o fallback quando o Redis falha e voltam ao Redis quando ele recupera. Cada escrita também vai para o fallback, para que esteja pronto quando o Redis falhar; um fallback `file` acrescenta uma escrita em disco a cada escrita no cache.

Contadores e locks nunca passam para o fallback, porque o fallback não os consegue partilhar com outros servidores:

- Os locks recusam por segurança: `get()` devolve `false` em vez de entregar um segundo lock não partilhado. Os locks tentam sempre o Redis, mesmo enquanto os valores são servidos pelo fallback, por isso defina um `timeout` curto na ligação Redis para que uma tentativa de lock não espere muito.
- `Cache::add()`, `increment()` e `decrement()` lançam `StoreOperationFailedException`. Os rate limiters usam-nos, por isso as rotas com limite falham durante a falha; consulte [contadores e locks durante uma falha](./limitations.md#contadores-e-locks-durante-uma-falha).

Prefira um fallback `file` ou `database`. Um fallback `array` vive na memória de cada worker e guarda uma cópia de cada chave escrita, incluindo as chaves guardadas para sempre.

O estado do failover vive em cada processo PHP. Um processo de longa duração (um worker Octane ou de filas) lembra-se das chaves que alterou durante a falha e invalida-as no Redis antes de voltar a usá-lo. Um processo que termina não o faz: com PHP-FPM, cada pedido é um processo, por isso um valor alterado ou apagado durante uma falha pode voltar a ser servido pelo Redis depois de este recuperar, até expirar. Se isso for importante, use TTLs curtos para esses dados ou execute `php artisan cache:clear` quando o Redis voltar. Consulte [stores resilientes](../guias/store-resiliente.md) para ver como funciona o circuit breaker.

## Cache em camadas

Manter os valores recentes na memória do processo, à frente da store partilhada:

```php
'cacheer' => [
    'driver' => 'cacheer',
    'backend' => 'redis',
    'connection' => 'cache',
    'l1' => 60,
],
```

Compensa em workers de longa duração, como Octane e workers de filas; com PHP-FPM, as cópias duram um pedido. `Cache::flush()` chega a todos os workers na sua verificação de coerência seguinte, no máximo cinco segundos depois. Uma alteração a uma única chave feita por outro worker é vista, no máximo, ao fim da idade `l1`. As cópias expiradas são removidas no máximo uma vez por minuto, por isso um worker não guarda todas as chaves que leu. Os locks leem sempre a store partilhada, nunca uma cópia local. Consulte [cache em camadas](../guias/cache-em-camadas.md).

## Criptografia e compressão

```php
'cacheer' => [
    'driver' => 'cacheer',
    'backend' => 'redis',
    'connection' => 'cache',
    'encryption' => [
        'active' => 'v1',
        'passphrases' => ['v1' => env('CACHEER_KEY')],
    ],
    'compress' => true,
],
```

Os valores são cifrados com AES-256-GCM e autenticados antes de serem lidos. As chaves não são cifradas, por isso não coloque segredos nas chaves de cache.

Use uma passphrase aleatória, não uma fácil de memorizar: a chave de cifra é o seu hash SHA-256, sem salt nem iterações, por isso uma passphrase previsível pode ser descoberta testando candidatas. Gere uma com:

```sh
php -r 'echo base64_encode(random_bytes(32)), PHP_EOL;'
```
 Para rodar a chave, adicione uma nova passphrase, torne-a `active` e mantenha a antiga até as suas entradas expirarem. Uma passphrase vazia, como um `CACHEER_KEY` por definir, é recusada quando a store é criada, em vez de ser usada. Consulte [criptografia e compressão](../guias/criptografia-e-compressao.md).

## Prefixo

Tal como as stores do Laravel, uma store `cacheer` usa a sua opção `prefix` ou, na falta dela, `cache.prefix`. O prefixo torna-se o espaço de chaves isolado da store (um [escopo](../guias/escopos.md) do CacheerPHP):

- `Cache::flush()` limpa apenas esse espaço de chaves. Outras aplicações e stores que partilhem o backend mantêm as suas entradas.
- A correspondência é literal: limpar `app_1` nunca afeta `app_10`.
- Os locks em uso mantêm-se. Os registos de lock ficam ao lado do espaço de chaves, por isso `Cache::flush()` e `php artisan cache:clear` durante um deploy não libertam os locks de jobs ou tarefas agendadas em execução.

Com um prefixo vazio, `Cache::flush()` limpa todo o espaço de chaves dessa store no backend, incluindo os locks.

Um prefixo não pode conter `/` nem caracteres de controlo, nem ter mais de 255 bytes. Um prefixo assim é recusado quando a store é criada.

## Relógio

As entradas expiram segundo o relógio do Laravel, por isso `$this->travel()` e `Carbon::setTestNow()` afetam-nas. Para fornecer um relógio próprio, registe `Silviooosilva\CacheerPhp\Contracts\Clock` no container.
