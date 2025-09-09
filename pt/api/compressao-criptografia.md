# Referência da API — Compressão & Criptografia

#### `useCompression()`
Ativa ou desativa a compressão dos dados antes do armazenamento. Quando ativada, os dados são serializados e comprimidos com `gzcompress`.

```php

<?php
$Cacheer->useCompression();        // habilita
$Cacheer->useCompression(false);   // desabilita
```

#### `useEncryption()`
Ativa criptografia usando AES-256-CBC. Forneça uma chave secreta para criptografar e descriptografar os dados em cache.

```php
$Cacheer->useEncryption('sua-chave-secreta');
```

É possível combinar ambos para payloads menores e seguros:

```php
$Cacheer->useCompression()->useEncryption('sua-chave-secreta');
```

