# Criptografia e compressão

Stores persistentes codificam valores por um pipeline que você descreve com um
`PipelineConfig`. Aqui está uma store que comprime e criptografa cada valor.

```php
use Silviooosilva\CacheerPhp\Kernel\Cache;
use Silviooosilva\CacheerPhp\Config\PipelineConfig;
use Silviooosilva\CacheerPhp\Storage\Encryption\Keyring;

$pipeline = PipelineConfig::default()
    ->withGzip()                                                     // comprime payloads grandes
    ->withKeyring(Keyring::fromPassphrases(['current' => $secret], 'current')); // AES-256-GCM

$cache = Cache::file(__DIR__ . '/cache', $pipeline);

$cache->set('token', 'sensitive-data');   // guardado comprimido + criptografado
echo $cache->get('token');                // 'sensitive-data' — descriptografado de forma transparente
```

- A criptografia é **autenticada**: dados adulterados ou truncados são rejeitados na
  leitura, nunca retornados. Requer `ext-openssl`.
- A compressão é **limitada**, então um blob criado com malícia não esgota a memória.
  Requer `ext-zlib`.
- Rotacione chaves adicionando um novo id e marcando-o como ativo — entradas antigas
  ainda descriptografam.

Veja o [guia de Criptografia e compressão](../guias/criptografia-e-compressao.md).
