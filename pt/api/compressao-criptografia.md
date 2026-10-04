# Compressão e criptografia — o envelope de armazenamento

Stores persistentes (`FileStore`, `DatabaseStore`, `RedisStore`) passam cada valor
por um pipeline e guardam o resultado em um **envelope versionado e autenticado**. O
pipeline é configurado com um [`PipelineConfig`](./configuracao.md).

## O envelope

```text
serialize  ->  comprime (opcional)  ->  criptografa (opcional)  ->  Envelope
```

O `Envelope` (`Silviooosilva\CacheerPhp\Storage\Envelope`) registra a versão do
formato, o id do serializer, do compressor, do encrypter e da chave, e então o
payload. É prefixado com NUL e um marcador mágico para nunca ser confundido com um
payload v5. O `EnvelopeCodec` codifica e decodifica; falhas são determinísticas e
tipadas — um blob adulterado, truncado, acima do limite ou não reconhecido lança
uma exceção em vez de retornar dados corrompidos ou não autenticados.

## Serializers

- `PhpSerializer` (padrão) — serialização nativa do PHP; lida com qualquer valor
  serializável, incluindo objetos.
- `JsonSerializer` — JSON portátil; lança para valores que o JSON não representa.

```php
$pipeline = PipelineConfig::default()->withJsonSerializer();
```

O `PhpSerializer` permite todas as classes por padrão, então sem criptografia o
backend é confiável: quem puder escrever nele pode plantar objetos que serão
desserializados na leitura. Se o backend for compartilhado, habilite a criptografia
ou restrinja as classes:

```php
use Silviooosilva\CacheerPhp\Storage\Serializer\PhpSerializer;

$pipeline = PipelineConfig::default()->withSerializer(new PhpSerializer(allowedClasses: false));
```

## Compressão

Gzip opcional, útil para payloads grandes. A descompressão é **limitada** (respeita
`withMaxValueBytes`), então um blob malicioso não força uso ilimitado de memória.

```php
$pipeline = PipelineConfig::default()->withGzip(level: 6);
```

## Criptografia — AES-256-GCM autenticado

A criptografia é autenticada: na leitura, ciphertext adulterado ou truncado é
rejeitado em vez de ser retornado. As chaves são gerenciadas por um `Keyring` que
suporta rotação.

```php
use Silviooosilva\CacheerPhp\Storage\Encryption\Keyring;

$keyring = Keyring::fromPassphrases(
    ['2025' => $oldSecret, '2026' => $newSecret],
    activeId: '2026',
);

$pipeline = PipelineConfig::default()->withKeyring($keyring);
```

- Novas gravações usam a chave **ativa**; o id da chave é guardado no envelope.
- Entradas antigas escritas com uma chave aposentada ainda descriptografam enquanto
  o id continuar no keyring — então você rotaciona sem flush.
- Um pipeline que criptografa lê **apenas** envelopes criptografados: um envelope em
  texto puro lança `UnsupportedEnvelopeException`, então ninguém contorna a
  autenticação gravando um envelope que declara "sem criptografia". Ao ativar a
  criptografia num cache existente, limpe-o antes (ou use um novo keyspace).
- Requer `ext-openssl`.

> Nunca cacheie segredos em uma store sem criptografia habilitada. O pipeline
> padrão não criptografa.

## Limites de tamanho

```php
$pipeline = PipelineConfig::default()->withMaxValueBytes(2_000_000);
```

Um valor cujo formato serializado excede o limite lança `ValueTooLargeException` na
escrita. O mesmo limite vale na leitura para todo pipeline — puro, comprimido ou
criptografado — antes de qualquer desserialização, e a descompressão para assim que
ele é excedido. `0` significa sem limite; um limite negativo lança
`InvalidArgumentException`.

## Dados que não são envelope

`decode()` aceita apenas envelopes v6. Qualquer outro blob — inclusive um payload
gravado pelo CacheerPHP v5 — lança `UnsupportedEnvelopeException` em vez de ser
decodificado, quaisquer que sejam os estágios do pipeline. A v6 não lê os dados em
cache da v5; veja o [guia de atualização](../atualizacao/index.md#5-dados-em-cache-a-v6-começa-fria).
