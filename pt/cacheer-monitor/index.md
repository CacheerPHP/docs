# Cacheer Monitor

O Cacheer Monitor é um dashboard local que observa um log de eventos JSONL emitido pelos clientes CacheerPHP. Visualiza tráfego de cache, latência, drivers e uso de chaves em tempo quase real, para depurar cargas de trabalho ou demonstrar o comportamento do Cacheer.

## O Que Faz

- Métricas ao vivo — hits, misses, puts, flushes e percentis de latência.
- Explorador do fluxo de eventos com filtros por namespace, tipo e chave.
- Gráfico de distribuição por driver, repartição por namespace e ranking das chaves mais usadas.
- Distribuição de TTL nos eventos de escrita (forever, > 1 dia, > 1 hora, etc.).
- Key Inspector — histórico de hit/miss por chave, último TTL, tamanho do valor e pré-visualização ao vivo.
- Exportação de eventos — descarregue o histórico em JSON ou CSV para análise offline.
- Captura de valores — gravação opcional dos valores em cache nos payloads dos eventos (oculta campos sensíveis automaticamente).
- Ações destrutivas protegidas por token para ambientes de desenvolvimento partilhados.
- Ingestão JSONL leve, sem dependências externas.

## Como Funciona

1. Os adaptadores do Cacheer acrescentam eventos a um ficheiro JSONL em disco.
2. O monitor expõe uma pequena API HTTP baseada nesse ficheiro.
3. A SPA consulta (e opcionalmente transmite via SSE) métricas e eventos.
4. Comandos CLI ajudam a gerir o servidor de desenvolvimento e os cenários de exemplo.

## Secções

- [Início Rápido](quick-start.md)
- [Referência da API](api.md)
- [Referência da CLI](cli.md)
- [Configuração](configuration.md)
- [Fluxos Comuns](workflows.md)
- [Resolução de Problemas](troubleshooting.md)
