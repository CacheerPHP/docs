# Cacheer Monitor

O Cacheer Monitor 2.x é um dashboard local para a telemetria do CacheerPHP 6. Mostra tráfego de cache registado, latência das operações, sinais do ciclo de vida e chaves problemáticas, para investigar o comportamento do cache enquanto a aplicação corre.

## O Que Faz

- Métricas ao vivo — hits, misses, puts, flushes e percentis de latência.
- Explorador do fluxo de eventos com filtros por namespace, tipo e chave.
- Ciclo de vida — valores antigos servidos, atualizações concluídas, promoções entre camadas e timeouts de espera por locks. Selecione um sinal para consultar os eventos correspondentes.
- Problem keys — as 10 chaves correspondentes com mais misses, erros, hits ou maior latência p95, agrupadas por driver e namespace reportado.
- Regras de saúde — limiares de taxa de acerto, erros e latência guardados no navegador, com mínimo de amostras, período de observação e snooze.
- Gráfico de distribuição por driver e repartições de namespace/TTL quando os metadados são reportados.
- Key Inspector — histórico por driver, contagens do ciclo de vida, metadados do valor e pré-visualização ao vivo opcional.
- Exportação de eventos — descarregue o histórico em JSON ou CSV para análise offline.
- Captura de valores — gravação opcional dos valores em cache nos payloads dos eventos (oculta campos sensíveis automaticamente).
- Ações destrutivas protegidas por token para ambientes de desenvolvimento partilhados.
- Ingestão JSONL leve, sem dependências externas.

## Como Funciona

1. O listener de telemetria v6 acrescenta eventos tipados a um ficheiro JSONL em disco.
2. O monitor expõe uma pequena API HTTP baseada nesse ficheiro.
3. O dashboard obtém um snapshot com métricas, um fluxo limitado de eventos, contagens de cobertura e dados dos gráficos. O SSE pede uma atualização quando chegam eventos e volta a ligar após timeouts.
4. Comandos CLI ajudam a gerir o servidor de desenvolvimento e os cenários de exemplo.

## Secções

- [Início Rápido](quick-start.md)
- [Referência da API](api.md)
- [Referência da CLI](cli.md)
- [Configuração](configuration.md)
- [Fluxos Comuns](workflows.md)
- [Resolução de Problemas](troubleshooting.md)

## O Que as Métricas Abrangem

As métricas de resumo usam todos os eventos do **log atual** que correspondem ao intervalo temporal e namespace selecionados. O limite do fluxo só altera o número de eventos apresentados. A pesquisa de chaves filtra o fluxo e os rankings; o filtro de operação afeta o fluxo. Nenhum deles altera as métricas de resumo ou as regras de saúde.

**All time** significa os registos retidos no ficheiro atual. Os logs arquivados não são incluídos. Os gráficos usam 20 intervalos na janela selecionada; com **All time**, mostram os últimos 10 minutos. As janelas temporais avançam em cada atualização.

A latência mede operações de cache registadas. Os sinais de ciclo de vida sem duração são excluídos. Sem amostras com duração, o dashboard mostra a latência como desconhecida.

A ponte atual da v6 não emite metadados de namespace ou TTL. O Monitor explica quando estes campos não estão disponíveis; a ausência de namespace não implica o escopo padrão, e a ausência de TTL não significa duração ilimitada. Eventos antigos ou personalizados podem continuar a fornecer estes campos.

Consulte os [Fluxos Comuns](workflows.md) para diagnósticos práticos e a [Configuração](configuration.md) para os valores padrão das regras de saúde.
