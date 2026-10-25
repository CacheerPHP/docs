# Cacheer Monitor

O Cacheer Monitor 2.x é um dashboard local para a telemetria do CacheerPHP 6. Mostra tráfego de cache registado, latência das operações, sinais do ciclo de vida e chaves problemáticas, para investigar o comportamento do cache enquanto a aplicação corre.

## Funcionalidades do dashboard

| Funcionalidade | Descrição |
| --- | --- |
| Cartões de métricas | Hits, misses, puts, flushes, renovações, incrementos, invalidações por tag, eliminações, erros e latência média/p95/p99. |
| Eficiência do cache | Taxa de acerto, número de consultas e indicador horizontal; a ausência de consultas é explícita. |
| Regras de saúde | Avisos por taxa de acerto, erros e latência p95, com mínimo de amostras, período de observação e snooze. |
| Gráficos | Hits/misses e latência média em 20 intervalos; All time mostra os últimos 10 minutos. |
| TTL e namespaces | Distribuições dos metadados reportados; os campos em falta são explicados. |
| Drivers | Distribuição do número de eventos por driver. |
| Problem keys | As 10 chaves correspondentes por ranking de hits, misses, erros ou p95, com escritas e amostras, agrupadas por driver e namespace. |
| Key patterns | Agrupamento por `{id}`, `{uuid}`, `{ulid}`, `{date}`, `{email}` e `{hash}`; rankings por taxa de acerto, tempo poupado, escritas desperdiçadas, misses, consultas e p95. |
| Trabalho desperdiçado | Escritas sobrescritas, apagadas ou expiradas sem leitura, e stampedes com misses, processos e execuções do loader. |
| Tempo poupado | Estimativa global e por padrão, com o número de execuções do loader que a sustenta (CacheerPHP 6.1+). |
| Ciclo de vida | Valores stale servidos, refreshes, promoções e timeouts de locks; selecione um sinal para filtrar eventos. |
| Failover | Fallback, circuitos abertos/fechados, reconciliação e janelas de falha, das mais recentes para as mais antigas (CacheerPHP 6.1+). |
| Fluxo de eventos | Tipo, chave, driver, duração, TTL, tamanho e tipo do valor. |
| Key Inspector | Histórico por driver, ciclo de vida, timestamps, metadados, linha temporal e pré-visualização capturada ou ao vivo, com ocultação de campos sensíveis. |
| Exportação | Eventos em JSON ou CSV. |
| SSE e atualização | Atualização por eventos e polling desativado, 1s, 2s, 5s ou 10s; a preferência fica em localStorage. |
| Filtro temporal | 5m, 15m, 1h, 6h, 24h ou All time. |
| Temas claro e escuro | Preferência persistente, sem flash do tema incorreto. |
| Indicador de atividade | Linha de progresso durante pedidos, respeitando movimento reduzido. |
| Navegação | Barra lateral persistente no desktop, navegação móvel compacta e secção ativa destacada. |
| Assets locais | Fontes, ícones SVG e Chart.js incluídos; não requer acesso a CDNs. |
| Teclado | Foco visível, botões acessíveis e inspector com contenção do foco e fecho por Escape. |
| Captura e proteção | Captura opcional de valores com redação de campos sensíveis e ações destrutivas protegidas por token. |

## Como Funciona

1. O listener de telemetria v6 acrescenta eventos tipados a um ficheiro JSONL em disco.
2. O monitor expõe uma pequena API HTTP baseada nesse ficheiro.
3. O dashboard obtém um snapshot com métricas, um fluxo limitado de eventos, contagens de cobertura e dados dos gráficos. O SSE pede uma atualização quando chegam eventos e volta a ligar após timeouts.
4. Comandos CLI iniciam o servidor local e diagnosticam a ponte de telemetria.

## Secções

- [Início Rápido](quick-start.md)
- [Referência da API](api.md)
- [Referência da CLI](cli.md)
- [Configuração](configuration.md)
- [Fluxos Comuns](workflows.md)
- [Resolução de Problemas](troubleshooting.md)
- [Contribuir](../contribuicao/cacheer-monitor.md)

## O Que as Métricas Abrangem

As métricas de resumo usam todos os eventos do **log atual** que correspondem ao intervalo temporal e namespace selecionados. O limite do fluxo só altera o número de eventos apresentados. A pesquisa de chaves filtra o fluxo e os rankings; o filtro de operação afeta o fluxo. Nenhum deles altera as métricas de resumo ou as regras de saúde.

**All time** significa os registos retidos no ficheiro atual. Os logs arquivados não são incluídos. Os gráficos usam 20 intervalos na janela selecionada; com **All time**, mostram os últimos 10 minutos. As janelas temporais avançam em cada atualização.

A latência mede operações de cache, não pedidos da aplicação, loaders ou consultas à base de dados. Os sinais sem duração, execuções de loaders e relatórios de failover são excluídos. Os rankings mostram o número de amostras com a média e p95; poucas amostras não dão uma estimativa estável. Sem amostras, a latência é desconhecida.

O CacheerPHP 6.1+ reporta namespace (escopo), TTL e operação; a versão 6.0 não. Namespace em falta é desconhecido, não o escopo predefinido; TTL em falta é desconhecido, não duração ilimitada. Um `ttl: null` explícito significa duração ilimitada. Relatórios de failover e tempos de loaders também requerem 6.1; consulte os [Fluxos Comuns](workflows.md).

Consulte os [Fluxos Comuns](workflows.md) para diagnósticos práticos e a [Configuração](configuration.md) para os valores padrão das regras de saúde.
