# Referência da API — Visão Geral

## **Classes Principais**

```bash
Silviooosilva\\CacheerPhp\\Cacheer
```

Classe principal do pacote, usada para todas as operações de cache.

## **Métodos**

### 1. **Configuração**

#### `setConfig()`
Inicia uma configuração personalizada do CacheerPHP.

[Referência — setConfig()](./configuracao.md)

### 2. **Drivers**
Permite definir os diferentes backends disponíveis para uso.

[Referência — setDriver()](./drivers.md)

### 3. **OptionBuilder**
O **OptionBuilder** simplifica a configuração com builders fluentes por driver:
- `OptionBuilder::forFile()` → Opções de arquivo (`dir`, `expirationTime`, `flushAfter`)
- `OptionBuilder::forRedis()` → Opções do Redis (`setNamespace`, `expirationTime` padrão, `flushAfter` auto‑flush)
- `OptionBuilder::forDatabase()` → Opções de BD (`table`, `expirationTime` padrão, `flushAfter` auto‑flush)

Notas:
- `expirationTime` funciona como TTL padrão quando você omite o TTL em `putCache()` (ou usa 3600). TTLs explícitos diferentes de 3600 prevalecem.
- `flushAfter` dispara uma verificação de limpeza automática ao inicializar o store; se o intervalo tiver passado, o store chama `flushCache()`.

[Referência — OptionBuilder](./construtor-de-opcoes.md)

### 4. **Compressão & Criptografia**
Métodos incorporados para reduzir espaço e proteger dados em cache.

[Referência — Compressão & Criptografia](./compressao-criptografia.md)

### 5. **Tags**
Agrupe chaves sob tags e invalide de forma eficiente entre drivers.

[Referência — Funções de Cache (tag/flushTag)](./funcoes-cache.md)

Veja um exemplo completo em: [Tutorial de Tagging](../tutoriais/exemplo-10-tagging.md)

