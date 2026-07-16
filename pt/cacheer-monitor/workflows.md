# Fluxos Comuns

Receitas práticas para depuração diária ou demonstrações.

## Repor o Ambiente de Testes

1. Execute `POST /api/events/clear` (ou clique em **Clear**).
2. Volte a correr um script de exemplo ou gere eventos a partir da sua aplicação.
3. Abra o dashboard e defina o intervalo de atualização para `1s` em demos ao vivo.

## Investigar uma Chave Lenta

1. Filtre os eventos por fragmento de chave no campo **filter key**.
2. Analise os indicadores de latência na linha temporal de eventos.
3. Cruze namespaces e drivers para detetar anomalias.

## Limitar o Dashboard a uma Janela Temporal

O cabeçalho tem botões rápidos de intervalo (**Last 5 min**, **Last 15 min**, **Last 1 h**, **Today**, **All**). Ao selecionar um, tanto os cartões de métricas como a lista de eventos ficam limitados a essa janela.

Também pode aplicar filtros programaticamente com os parâmetros `from`/`until` da API (timestamps Unix):

```bash
NOW=$(date +%s)
curl "http://127.0.0.1:9966/api/metrics?from=$((NOW - 900))&until=$NOW"
```

## Analisar uma Chave a Fundo com o Key Inspector

O painel Key Inspector mostra o histórico de hit/miss por chave, o último TTL, o tamanho e tipo do valor e (com a captura de valores ativa) uma pré-visualização ao vivo do valor em cache.

1. Clique em qualquer chave na lista **Recent Events** ou no ranking **Top Keys**.
2. O painel lateral mostra o resumo da chave e todo o seu histórico de eventos.
3. Para forçar uma leitura ao vivo do cache (e não apenas do log), clique em **Refresh Live** dentro do inspetor. Requer `CACHEER_MONITOR_CAPTURE_VALUES=true`.

## Exportar o Histórico de Eventos

Descarregue um snapshot dos eventos registados para análise offline ou arquivo:

```bash
# Descarregar em JSON
curl "http://127.0.0.1:9966/api/events/export?format=json" -O -J

# Descarregar em CSV
curl "http://127.0.0.1:9966/api/events/export?format=csv" -O -J

# Limitado a um namespace e a uma janela temporal
curl "http://127.0.0.1:9966/api/events/export?format=csv&namespace=critical&from=1714300000&until=1714400000" -O -J
```

O CSV inclui as colunas: `ts`, `type`, `key`, `namespace`, `driver`, `duration_ms`, `success`, `size_bytes`, `ttl`.

## Limpar Arquivos Rodados

Cada `POST /api/events/clear` arquiva o log atual com um sufixo de timestamp. Remova arquivos antigos automaticamente:

```bash
curl -X POST http://127.0.0.1:9966/api/events/cleanup-rotated \
  -H "Content-Type: application/json" \
  -d '{"max_age_days": 7}'
```

## Vigiar a Taxa de Acerto com o Banner de Alerta

O dashboard inclui um limiar configurável de taxa de acerto. Quando a taxa global cai abaixo do limiar, um banner de alerta vermelho aparece no topo da página.

1. Introduza uma percentagem (por exemplo, `80`) no campo **Alert Threshold %** do cabeçalho.
2. O banner aparece automaticamente sempre que a taxa de acerto atual estiver abaixo desse valor.
3. Defina o valor como `0` para desativar o alerta.

É útil para apanhar regressões de cache durante testes de carga ou após um deploy — deixe o dashboard aberto e o banner dispara assim que a taxa de acerto degradar.

## Comparar Namespaces

Use o filtro de namespace para isolar tráfego. Combine com o gráfico circular e os cartões de resumo para quantificar o volume por namespace. Exporte capturas de ecrã para relatórios.

## Automatizar Relatórios

Chame o endpoint `/api/metrics` a partir do CI para capturar orçamentos de regressão, ou envie snapshots JSONL para pipelines de análise para comparações de longo prazo.

```bash
# Métricas da última hora
NOW=$(date +%s)
HOUR_AGO=$((NOW - 3600))
curl "http://127.0.0.1:9966/api/metrics?from=$HOUR_AGO&until=$NOW"
```
