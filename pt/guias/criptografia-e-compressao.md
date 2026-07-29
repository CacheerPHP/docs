# Criptografia e compressão

Stores persistentes codificam cada valor por um pipeline —
**serialize → comprime → criptografa** — e guardam o resultado em um envelope
versionado e autenticado. Você descreve o pipeline com um
[`PipelineConfig`](../api/configuracao.md) imutável e o passa para a store.

```php
use Silviooosilva\CacheerPhp\Config\PipelineConfig;
use Silviooosilva\CacheerPhp\Storage\Encryption\Keyring;
use Silviooosilva\CacheerPhp\Kernel\Cache;

$pipeline = PipelineConfig::default()
    ->withGzip()
    ->withKeyring(Keyring::fromPassphrases(['current' => $secret], 'current'));

$cache = Cache::file('/var/cache/app', $pipeline);
```

## Compressão

Gzip é opcional e mais útil para payloads grandes (arrays grandes, fragmentos HTML). A
descompressão é **limitada** por `withMaxValueBytes()`, então um blob criado com
malícia não força uso ilimitado de memória.

```php
$pipeline = PipelineConfig::default()->withGzip(level: 6)->withMaxValueBytes(2_000_000);
```

## Criptografia — AES-256-GCM autenticado

A criptografia é **autenticada**: na leitura, ciphertext adulterado ou truncado é
rejeitado com uma exceção tipada — nunca é retornado como dado. Requer `ext-openssl`.

```php
$keyring = Keyring::fromPassphrases(['2026' => $secret], activeId: '2026');
$pipeline = PipelineConfig::default()->withKeyring($keyring);
```

### Rotação de chaves

O envelope registra qual id de chave criptografou cada valor. Para rotacionar, adicione
uma nova chave, torne-a ativa e mantenha o id antigo disponível para leituras:

```php
$keyring = Keyring::fromPassphrases(
    ['2025' => $oldSecret, '2026' => $newSecret],
    activeId: '2026',   // novas gravações usam 2026
);
```

Entradas antigas escritas sob `2025` continuam descriptografando; novas gravações usam
`2026`. Sem flush, sem downtime.

## A ordem importa

O pipeline comprime **antes** de criptografar (dado criptografado não comprime). O
CacheerPHP cuida da ordem para você — você só declara os estágios.

## Combinando com um tamanho máximo

```php
$pipeline = PipelineConfig::default()
    ->withGzip()
    ->withKeyring($keyring)
    ->withMaxValueBytes(1_000_000); // ValueTooLargeException em escritas grandes demais
```

## Garantias e limites

- Adulteração, chave errada, truncamento ou um valor acima do limite produzem
  **falhas determinísticas e tipadas** — nunca corrupção silenciosa ou dado não
  autenticado.
- O pipeline padrão **não** criptografa nem comprime. Opte explicitamente; nunca
  cacheie segredos sem criptografia.
- Veja a [referência de Compressão e criptografia](../api/compressao-criptografia.md)
  para o formato do envelope e a compatibilidade de leitura v5.
