# Referência da API

Resumo das principais APIs e links para os tópicos detalhados.

- [Configuração: `setConfig()`](./configuracao.md)
- [Seleção de driver: `setDriver()`](./drivers.md)
- [Opções fluentes: `OptionBuilder`](./construtor-de-opcoes.md)
- [Auxiliar de tempo: `TimeBuilder`](./construtor-de-tempo.md)
- [Compressão e criptografia](./compressao-criptografia.md)
- [Funções de cache (get/put/has/flush/etc.)](./funcoes-cache.md)

Notas
- `expirationTime` funciona como TTL padrão quando você não informa na `putCache()` (ou usa o padrão 3600). TTLs explícitos diferentes de 3600 prevalecem.
- `flushAfter` habilita uma verificação de limpeza automática ao inicializar o store; se o intervalo tiver passado, o store chama `flushCache()`.

Se preferir uma visão em página única, consulte [Visão Geral (PT)](./visao-geral.md) (índice legado).
