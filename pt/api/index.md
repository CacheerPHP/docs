# Referência da API

Resumo das principais APIs e links para os tópicos detalhados.

- [Configuração: `setConfig()`](../../en/api/config.md)
- [Seleção de driver: `setDriver()`](../../en/api/drivers.md)
- [Opções fluentes: `OptionBuilder`](../../en/api/option-builder.md)
- [Auxiliar de tempo: `TimeBuilder`](../../en/api/time-builder.md)
- [Compressão e criptografia](../../en/api/compression-encryption.md)
- [Funções de cache (get/put/has/flush/etc.)](./funcoes-cache.md)

Notas
- `expirationTime` funciona como TTL padrão quando você não informa na `putCache()` (ou usa o padrão 3600). TTLs explícitos diferentes de 3600 prevalecem.
- `flushAfter` habilita uma verificação de limpeza automática ao inicializar o store; se o intervalo tiver passado, o store chama `flushCache()`.

Se preferir uma visão em página única, consulte [Visão Geral (EN)](../../en/api/overview.md) (índice legado).
