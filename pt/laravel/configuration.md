# Configurar o CacheerPHP para Laravel

Cada store `cacheer` é uma entrada em `config/cache.php`. Pode definir várias, por exemplo uma store Redis como predefinida e uma store de ficheiros para dados locais.

## Opções

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
| `l1` | nenhum | Manter cópias no processo à frente da store: um número de segundos (a idade máxima) ou `true` para 60. |
| `encryption` | nenhum | `['active' => id, 'passphrases' => [id => passphrase]]`. |
| `compress` | `false` | `true`, ou um nível de gzip de 1 a 9. |
| `max_value_bytes` | nenhum | Rejeitar valores maiores na escrita e na leitura. |

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

As leituras e escritas passam para o fallback quando o Redis falha e voltam ao Redis quando ele recupera. Contadores e locks nunca passam para o fallback: durante a falha, recusam por segurança (o lock não é adquirido) em vez de entregarem um segundo lock não partilhado.

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

Compensa em workers de longa duração, como Octane e workers de filas; com PHP-FPM, as cópias duram um pedido. `Cache::flush()` chega a todos os workers na sua verificação de coerência seguinte, no máximo cinco segundos depois. Uma alteração a uma única chave feita por outro worker é vista, no máximo, ao fim da idade `l1`. Os locks leem sempre a store partilhada, nunca uma cópia local. Consulte [cache em camadas](../guias/cache-em-camadas.md).

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

Os valores são cifrados com AES-256-GCM e autenticados antes de serem lidos. As chaves não são cifradas, por isso não coloque segredos nas chaves de cache. Para rodar a chave, adicione uma nova passphrase, torne-a `active` e mantenha a antiga até as suas entradas expirarem. Uma passphrase vazia, como um `CACHEER_KEY` por definir, é recusada quando a store é criada, em vez de ser usada. Consulte [criptografia e compressão](../guias/criptografia-e-compressao.md).

## Prefixo

Tal como as stores do Laravel, uma store `cacheer` usa a sua opção `prefix` ou, na falta dela, `cache.prefix`. O prefixo torna-se o espaço de chaves isolado da store (um [escopo](../guias/escopos.md) do CacheerPHP):

- `Cache::flush()` limpa apenas esse espaço de chaves. Outras aplicações e stores que partilhem o backend mantêm as suas entradas.
- A correspondência é literal: limpar `app_1` nunca afeta `app_10`.

Com um prefixo vazio, `Cache::flush()` limpa todo o espaço de chaves dessa store no backend.

## Relógio

As entradas expiram segundo o relógio do Laravel, por isso `$this->travel()` e `Carbon::setTestNow()` afetam-nas. Para fornecer um relógio próprio, registe `Silviooosilva\CacheerPhp\Contracts\Clock` no container.
