# Fluxos Comuns

Receitas práticas para depuração diária ou demonstrações.

## Repor o Ambiente de Testes

1. Execute `POST /api/events/clear` (ou abra **Log maintenance** e clique em **Clear events**).
2. Volte a correr um script de exemplo ou gere eventos a partir da sua aplicação.
3. Abra o dashboard e defina o intervalo de atualização para `1s` em demos ao vivo.

## Investigar uma Chave Lenta

1. Abra **Problem keys** em **Cache explorer** e escolha **Highest p95 latency** em **Rank by**.
2. Compare a latência média/p95 e o número de amostras com duração. Uma única amostra lenta não é uma estimativa estável.
3. Use **Search keys…** para limitar os rankings e o fluxo de eventos. A pesquisa é aplicada antes de selecionar as 10 primeiras chaves.
4. Clique numa chave para consultar o histórico desse driver e namespace reportado.

A latência mede operações de cache, não a duração dos pedidos nem o tempo de cálculo de um valor. Sinais de ciclo de vida sem duração são excluídos.

## Encontrar Chaves com Misses ou Erros

Escolha **Most misses** ou **Most errors** em **Problem keys**. As chaves sem ocorrências para esse ranking são omitidas; uma chave com apenas misses pode aparecer. A tabela também mostra hits, taxa de acerto, escritas e amostras com duração. **Most hits** apresenta o ranking de atividade.

Chaves com o mesmo texto em drivers ou namespaces reportados diferentes continuam separadas. Um namespace marcado como **unreported** não é o escopo padrão.

## Investigar os Sinais do Ciclo de Vida

O painel **Cache lifecycle** conta os sinais registados no intervalo atual:

| Sinal | Significado |
|---|---|
| **Stale served** | Foi devolvido um valor mais antigo do cache. |
| **Refreshes** | O trabalho de atualização foi concluído. |
| **Promotions** | Uma store em camadas promoveu um valor para a camada mais rápida. |
| **Lock contention** | A espera por um lock de single-flight atingiu o timeout. Não conta todas as esperas por locks. |

Selecione um sinal para filtrar o fluxo de eventos e clique numa chave para abrir o inspetor. Este mostra contagens do ciclo de vida e uma linha temporal para essa chave e driver. Limpe a pesquisa de chaves se ocultar eventos que esperava encontrar.

## Limitar o Dashboard a uma Janela Temporal

O cabeçalho tem **5m**, **15m**, **1h**, **6h**, **24h** e **All time**. Uma janela temporal selecionada avança em cada atualização e aplica-se às métricas de resumo, contagens do ciclo de vida, rankings e fluxo de eventos. Os gráficos usam 20 intervalos nessa janela; com **All time**, mostram os últimos 10 minutos.

As métricas incluem todos os registos correspondentes do log atual, independentemente do limite do fluxo. Os ficheiros arquivados são excluídos. Os filtros de chave e operação limitam o fluxo sem alterar as métricas de resumo ou as regras de saúde; a pesquisa de chaves também limita os rankings.

Também pode aplicar filtros programaticamente com os parâmetros `from`/`until` da API (timestamps Unix):

```bash
NOW=$(date +%s)
curl "http://127.0.0.1:9966/api/snapshot?limit=50&from=$((NOW - 900))&until=$NOW"
```

## Analisar uma Chave a Fundo com o Key Inspector

O Key Inspector mostra hits, misses, escritas, taxa de acerto, contagens do ciclo de vida, metadados do valor e os últimos 15 eventos, dos mais recentes para os mais antigos. O resumo abrange o histórico correspondente da chave no log atual, independentemente da janela temporal do dashboard.

1. Clique numa chave em **Event stream** ou **Problem keys**.
2. Consulte as contagens do ciclo de vida e a linha temporal para o driver e os metadados de namespace selecionados.
3. Para pedir uma leitura ao vivo do valor, use o ícone de atualização do inspetor (**Refresh live value**). Requer `CACHEER_MONITOR_CAPTURE_VALUES=true`; a pré-visualização ao vivo só está disponível quando o processo do Monitor consegue resolver a store.

O CacheerPHP 6.1+ reporta TTL e namespace; a versão 6.0 não. **Not reported** indica metadados em falta, não uma entrada que nunca expira.

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

## Configurar as Regras de Saúde do Dashboard

1. Abra **Health rules**.
2. Mantenha **Low hit-rate warning** ativo e escolha **Alert below** no cartão de eficiência, ou ative **Error-count warning** e **p95 latency warning** com os respetivos limiares.
3. Defina **Minimum samples**, **Condition holds for** e **Snooze / repeat cooldown**. Consulte a [Configuração](configuration.md) para os valores padrão e o tipo de amostra usado por cada regra.
4. Deixe o dashboard aberto. As condições são avaliadas em atualizações bem-sucedidas, para o intervalo temporal e namespace atuais.
5. Use **Snooze** para ocultar um aviso durante o cooldown. Desative uma regra na respetiva caixa de seleção.

As definições ficam guardadas neste navegador. Os avisos permanecem visíveis até a condição recuperar ou usar snooze; se voltar a ocorrer após recuperar, o aviso respeita o cooldown de repetição. Alterar definições, intervalo temporal ou namespace, recarregar a página ou perder a ligação inicia uma nova avaliação. Não há notificações em segundo plano com o dashboard fechado.

## Comparar Namespaces

Use o filtro de namespace quando os eventos reportam explicitamente esse campo. Combine-o com o gráfico de drivers e os cartões de resumo para comparar tráfego registado. A ponte do CacheerPHP 6.0 não fornece metadados de namespace; o filtro indica que não está disponível quando os registos apresentados não os incluem. O filtro `(default)` corresponde a registos explicitamente sem escopo, não a registos com metadados em falta.

## Automatizar Relatórios

Consulte `/api/metrics` para criar um relatório ou arquive exportações JSONL para comparações ao longo do tempo. Use `limit=0` para agregar todos os registos correspondentes no log atual; por omissão, `/api/metrics` limita a agregação a 1.000 eventos.

```bash
# Métricas da última hora
NOW=$(date +%s)
HOUR_AGO=$((NOW - 3600))
curl "http://127.0.0.1:9966/api/metrics?limit=0&from=$HOUR_AGO&until=$NOW"
```

## Comparar padrões de chaves

Abra **Key patterns** para agrupar chaves que diferem num identificador, como
`user:123` e `user:456`, que passam a `user:{id}`. Os formatos incluem `{id}`,
`{uuid}`, `{ulid}`, `{date}`, `{email}` e `{hash}`, por driver e namespace.
Pode ordenar pela menor taxa de acerto (apenas padrões com pelo menos 20
consultas), tempo poupado, escritas desperdiçadas, misses, consultas ou p95.
Use a pesquisa de chaves para restringir os rankings.

## Investigar failover

O painel **Failover** requer CacheerPHP 6.1 e um cache resiliente. O estado vive
em cada processo PHP; um pedido PHP-FPM curto costuma usar o fallback sem abrir
o circuito. Por isso, as janelas de falha vêm da atividade no fallback: uma
separação superior a um minuto inicia outra janela.

Uma janela mostra **Recovery recorded** quando se segue um fecho de circuito ou
uma reconciliação. Caso contrário, mostra **No recovery recorded yet**, o que
não significa que a falha ainda esteja a decorrer.

## Estimar tempo poupado

O CacheerPHP 6.1 reporta a duração do loader num miss de `remember()` ou
`flexible()`. Para cada padrão, o Monitor multiplica os hits pela diferença
entre o tempo médio do loader e o tempo médio de um hit, assumindo que cada hit
teria executado o loader uma vez. É uma estimativa.

Padrões escritos com `set()` não têm tempos de loader; a poupança fica
desconhecida, em vez de zero, e não entra no total. O painel mostra também o
número de execuções em que baseia a estimativa.

## Encontrar escritas desperdiçadas

Uma escrita de valor (`put`, escrita sem expiração, lote ou `add`) é desperdiçada
se foi sobrescrita, apagada ou expirou antes de qualquer hit a ler. A expiração
só conta quando o TTL foi reportado e terminou antes do fim do log atual.

Uma escrita que ainda pode estar válida, ou cujo TTL é desconhecido, não tem
resultado conhecido e não é contada em nenhum dos lados. Flushes da store não
são atribuídos a escritas individuais. Contadores, renovações e tags não são
escritas de valores.

## Investigar stampedes

Um **stampede** é um conjunto de pelo menos 3 misses da mesma chave em 2
segundos, vindos de pelo menos 2 processos. Com CacheerPHP 6.1, o Monitor também
conta as execuções do loader nessa rajada. O single-flight de `remember()` evita
a maioria destes casos quando a store suporta locks; investigue stores sem
locks e loaders mais lentos do que o tempo de espera pelo lock.

O `ArrayStore` vive num único processo. As suas escritas e misses são avaliadas
por processo: outro processo com a mesma chave não usa a mesma cópia.
