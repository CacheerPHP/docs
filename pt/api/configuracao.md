# Configuração

A v6 não tem configuração ambiente — sem `.env`, sem timezone global, sem schema
implícito. Um `Cache` é exatamente o que você constrói. Duas coisas são
configuradas: **como a store é construída** (construtores nomeados) e **como os
valores são armazenados** (`PipelineConfig`).

## Construtores nomeados

O caminho comum precisa de uma linha:

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;

$cache = Cache::inMemory();                    // ArrayStore
$cache = Cache::file('/var/cache/app');        // FileStore
$cache = Cache::database($pdo, 'cacheer');     // DatabaseStore (injete o PDO)
$cache = Cache::redis($connection);            // RedisStore (injete a conexão)
```

Cada construtor persistente aceita um `PipelineConfig` e `Clock` opcionais. Veja
[Cache e ScopedCache](./funcoes-cache.md#construtores-nomeados) para as assinaturas
completas, incluindo os decorators `tiered`, `resilient` e `instrumented`.

## `PipelineConfig` — o pipeline de armazenamento

`Silviooosilva\CacheerPhp\Config\PipelineConfig` é uma descrição imutável de como um
valor vira bytes: **serialize → (comprime) → (criptografa)**. Cada `with*()` retorna
uma nova instância.

```php
use Silviooosilva\CacheerPhp\Config\PipelineConfig;
use Silviooosilva\CacheerPhp\Storage\Encryption\Keyring;
use Silviooosilva\CacheerPhp\Storage\Compat\V5PayloadReader;

$pipeline = PipelineConfig::default()          // serializer PHP, sem compressão/criptografia
    ->withJsonSerializer()
    ->withGzip(level: 6)
    ->withKeyring(Keyring::fromPassphrases(['current' => $secret], 'current')) // AES-256-GCM
    ->withMaxValueBytes(2_000_000)
    ->withV5Reader(new V5PayloadReader());

$cache = Cache::file('/var/cache/app', $pipeline);
```

| Método | Efeito |
|---|---|
| `default()` | Serialização PHP, sem compressão nem criptografia |
| `withSerializer(Serializer)` / `withJsonSerializer()` | Escolhe o serializer |
| `withCompressor(Compressor)` / `withGzip(int $level = 6)` | Adiciona compressão |
| `withEncrypter(Encrypter)` / `withKeyring(Keyring)` | Adiciona criptografia autenticada |
| `withMaxValueBytes(int)` | Impõe um tamanho serializado máximo na escrita |
| `withV5Reader(V5PayloadReader)` | Habilita leitura de payloads v5 |
| `codec()` | Constrói o `EnvelopeCodec` pronto (as stores chamam isto) |

O pipeline é detalhado em [Compressão e criptografia](./compressao-criptografia.md) e
no [guia de criptografia e compressão](../guias/criptografia-e-compressao.md).

## Injetando suas próprias dependências

Para controle total, construa `Cache` diretamente:

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Support\SystemClock;

$cache = new Cache(
    store:    $store,
    clock:    new SystemClock(),
    executor: $deferredExecutor,   // para stale refresh após a resposta
    events:   $eventDispatcher,    // observabilidade
);
```

A leitura de variáveis de ambiente pertence à sua aplicação ou a uma ponte
opcional — não à biblioteca.
