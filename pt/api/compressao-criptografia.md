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
- Requer `ext-openssl`.

> Nunca cacheie segredos em uma store sem criptografia habilitada. O pipeline
> padrão não criptografa.

## Limites de tamanho

```php
$pipeline = PipelineConfig::default()->withMaxValueBytes(2_000_000);
```

Um valor cujo formato serializado excede o limite lança `ValueTooLargeException` na
escrita, e o mesmo limite delimita a descompressão na leitura.

## Lendo dados v5

Durante uma migração você pode ler valores escritos pela v5 anexando um
`V5PayloadReader` que espelhe a compressão/criptografia usada no seu app v5 (payloads
v5 não são autodescritivos):

```php
use Silviooosilva\CacheerPhp\Storage\Compat\V5PayloadReader;

$pipeline = PipelineConfig::default()->withV5Reader(new V5PayloadReader(compression: true));
```

`FileStore` e `DatabaseStore` podem ainda **reescrever** valores antigos no envelope
v6 na leitura (`migrateLegacyOnRead: true`). Veja o
[guia de atualização](../atualizacao/index.md#5-compatibilidade-de-dados-e-reescrita-na-leitura).
A v5 usava AES-256-CBC não autenticado — uma chave errada aparece apenas como falha
de `unserialize`, nunca criptograficamente.
