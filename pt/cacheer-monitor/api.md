# Referência da API

Endpoints REST alimentam a SPA e oferecem uma superfície amigável para ferramentas e scripts.

## Endpoints REST

### `GET /api/health`

Sonda simples de disponibilidade. Devolve `{ "ok": true }` quando o servidor está acessível.

---

### `GET /api/config`

Indica o caminho do ficheiro de eventos, como foi resolvido (`env`, `dotenv`, `default`) e outras informações de runtime.

```json
{
  "events_file": "/tmp/cacheer-monitor.jsonl",
  "origin": "env"
}
```

---

### `GET /api/metrics`

Agrega registos correspondentes do log atual em contadores, taxa de acerto, percentis de latência, contagens do ciclo de vida e rankings de chaves. O limite padrão é de 1.000 registos; use `limit=0` para agregar todos os correspondentes. Os logs arquivados são excluídos.

| Parâmetro | Padrão | Descrição |
|---|---|---|
| `namespace` | — | Filtra por namespace explicitamente reportado. `(default)` corresponde a um namespace explicitamente vazio, não a metadados em falta. |
| `limit` | `1000` | Número máximo de registos correspondentes a agregar. `0` significa todos. |
| `from` | — | Timestamp Unix (float). Inclui apenas eventos a partir deste instante. |
| `until` | — | Timestamp Unix (float). Inclui apenas eventos até este instante. |

Exemplo (campos selecionados):

```json
{
  "hits": 2,
  "misses": 1,
  "puts": 1,
  "errors": 0,
  "total_events": 7,
  "latency": { "avg_ms": 2, "p95_ms": 3.7, "p99_ms": 3.94 },
  "latency_samples": 4,
  "namespace_samples": 0,
  "ttl_samples": 0,
  "lifecycle": {
    "stale_served": 1,
    "refresh": 1,
    "promotion": 0,
    "lock_contended": 1
  }
}
```

`latency_samples` conta operações de cache com duração. Os sinais de ciclo de vida sem duração são excluídos; com contagem zero, os valores de latência são zeros de preenchimento e o dashboard apresenta `—`. `namespace_samples` e `ttl_samples` contam registos com metadados disponíveis. Um mapa de namespaces, por si só, não comprova que foram reportados metadados de escopo. TTL em falta não incrementa o grupo de duração ilimitada.

`top_keys` continua a ser um mapa antigo de contagens de hits. O dashboard usa `problem_keys`: quatro arrays chamados `misses`, `errors`, `hits` e `latency`, cada um com até 10 linhas ordenadas por contagem ou latência p95 decrescente. Só são incluídas linhas com ocorrências para esse ranking. Cada linha é agrupada por chave, driver e metadados de namespace.

Exemplo de linha de diagnóstico:

```json
{
  "key": "users:42",
  "driver": "FileStore",
  "namespace": null,
  "hits": 2,
  "misses": 1,
  "errors": 0,
  "writes": 1,
  "events": 7,
  "hit_rate": 0.6666666666666666,
  "latency": { "avg_ms": 2, "p95_ms": 3.7, "p99_ms": 3.94 },
  "latency_samples": 4
}
```

Aqui, `namespace: null` significa não reportado; `namespace: ""` significa explicitamente sem escopo. `hit_rate` é `null` para chaves sem consultas. A latência mede operações de cache, não o tempo de execução de loaders, consultas à base de dados ou pedidos.

---

### `GET /api/snapshot`

Devolve métricas, fluxo de eventos, contagens de cobertura e dados dos gráficos numa leitura do log atual. Ao contrário de `/api/metrics`, o seu `limit` aplica-se apenas ao fluxo; as métricas e gráficos usam todos os registos que correspondem ao intervalo temporal e namespace.

| Parâmetro | Padrão | Descrição |
|---|---|---|
| `limit` | `1000` | Máximo de eventos no fluxo. `0` devolve todos os correspondentes. O dashboard começa em `200`. |
| `namespace` | — | Filtra o snapshot por namespace explicitamente reportado. |
| `from` | — | Timestamp Unix (float), limite inferior inclusivo. |
| `until` | — | Timestamp Unix (float), limite superior inclusivo. |
| `key_filter` | Vazio | Fragmento de chave sem distinção entre maiúsculas e minúsculas; filtra o fluxo e os rankings antes dos respetivos limites. |
| `type` | Vazio | Tipo exato de evento, como `lock_contended`; filtra o fluxo antes do limite. |

Os filtros de chave/tipo não alteram as métricas de resumo, contagens do ciclo de vida ou dados da linha temporal. A pesquisa de chaves só altera os rankings e o fluxo; o tipo só altera o fluxo.

```bash
curl "http://127.0.0.1:9966/api/snapshot?limit=50&key_filter=users&from=1714310000&until=1714310600"
```

| Campo da resposta | Conteúdo |
|---|---|
| `metrics` | Campos agregados descritos em `/api/metrics`, incluindo contagens do ciclo de vida e rankings de diagnóstico. |
| `events` | Registos correspondentes mais recentes, na ordem do ficheiro (mais antigos primeiro). A UI inverte-os para apresentação. |
| `coverage.source` | `current_log`; os ficheiros arquivados são excluídos. |
| `coverage.matching_events` | Número de registos usados nas métricas após filtrar por namespace/tempo. |
| `coverage.matching_feed_events` | Número restante após filtrar por chave/tipo, antes do limite do fluxo. |
| `coverage.shown_events` | Número devolvido em `events`. |
| `timeline` | `from`, `until`, `interval_seconds` e 20 `buckets`. Cada intervalo tem `ts`, `hits`, `misses`, `samples` e `avg_ms`; a latência média é `null` sem amostras com duração. |

Com limites temporais explícitos, a linha temporal usa esse intervalo; cada intervalo de agregação tem pelo menos um segundo. Sem `from`, cobre os últimos 10 minutos até `until` ou ao instante atual. O dashboard avança os limites das janelas em cada atualização; os timestamps da API são limites fixos fornecidos por quem a chama.

---

### `GET /api/events`

Devolve um array JSON dos registos correspondentes mais recentes, na ordem do ficheiro, dos mais antigos para os mais recentes. Os filtros de namespace/tempo são aplicados antes do limite. Use `/api/snapshot` para filtrar por fragmento de chave ou tipo de evento.

| Parâmetro | Padrão | Descrição |
|---|---|---|
| `limit` | `200` | Número máximo de eventos devolvidos. |
| `namespace` | — | Restringe a um único namespace. |
| `from` | — | Timestamp Unix (float). Inclui apenas eventos a partir deste instante. |
| `until` | — | Timestamp Unix (float). Inclui apenas eventos até este instante. |

```json
[
  {
    "type": "put",
    "ts": 1714310557,
    "instance": "a1b2c3d4",
    "payload": {
      "key": "users:42",
      "driver": "FileStore",
      "duration_ms": 4,
      "success": true
    }
  }
]
```

---

### `GET /api/keys/inspect`

Devolve um resumo por chave e os registos recentes, dos mais recentes para os mais antigos. O resumo abrange o histórico correspondente da chave no log atual, independentemente de `limit` e da janela temporal do dashboard. A UI apresenta os últimos 15 eventos.

| Parâmetro | Padrão | Descrição |
|---|---|---|
| `key` | **obrigatório** | A chave exata a inspecionar. |
| `namespace` | — | Restringe a um namespace específico. |
| `driver` | — | Restringe o histórico a um driver reportado, por exemplo `FileStore`. |
| `namespace_missing` | `false` | Quando `true`, só inclui registos sem o campo namespace. Use para uma linha de diagnóstico com namespace não reportado. |
| `limit` | `100` | Número máximo de eventos da chave devolvidos. |
| `live` | `false` | Quando `true` e a captura de valores está ativa, força uma leitura ao vivo do cache para preencher a pré-visualização. |

Exemplo para `key=users:42&driver=FileStore&namespace_missing=1&limit=1` (campos selecionados do resumo):

```json
{
  "summary": {
    "key": "users:42",
    "hits": 2,
    "misses": 1,
    "puts": 1,
    "last_ttl": null,
    "last_ttl_known": false,
    "namespace_samples": 0,
    "latency_samples": 4,
    "lifecycle": { "stale_served": 1, "refresh": 1, "promotion": 0, "lock_contended": 1 },
    "capture_values_enabled": false,
    "drivers": { "FileStore": 7 }
  },
  "events": [
    {
      "type": "lock_contended",
      "ts": 1714310562,
      "instance": "a1b2c3d4",
      "payload": { "key": "users:42", "driver": "FileStore", "duration_ms": 0, "success": true }
    }
  ]
}
```

`last_ttl_known` distingue TTL indisponível de duração ilimitada explicitamente reportada (`last_ttl: null` com `last_ttl_known: true`). Os eventos atuais da v6 não fornecem campos de TTL ou namespace. O resumo também inclui erros, estatísticas de latência, metadados do valor e informação de pré-visualização opcional.

---

### `GET /api/events/export`

Descarrega um snapshot dos eventos guardados em JSON ou CSV.

| Parâmetro | Padrão | Descrição |
|---|---|---|
| `format` | `json` | `json` ou `csv`. |
| `limit` | `0` (todos) | Número máximo de eventos a exportar. `0` significa sem limite. |
| `namespace` | — | Restringe a um único namespace. |
| `from` | — | Timestamp Unix (float). |
| `until` | — | Timestamp Unix (float). |

A resposta inclui os cabeçalhos `Content-Disposition` e `Content-Type` adequados para download direto.

Colunas do CSV: `ts`, `type`, `key`, `namespace`, `driver`, `duration_ms`, `success`, `size_bytes`, `ttl`.

---

### `POST /api/events/clear`

> **Destrutivo.** Use apenas em ambientes locais/de desenvolvimento.

Roda o ficheiro de eventos: o ficheiro atual é arquivado com um sufixo de timestamp e é criado um novo ficheiro vazio. O botão **Clear** do dashboard usa este endpoint.

Devolve `{ "ok": true }`.

Se `CACHEER_MONITOR_TOKEN` estiver definido, este endpoint exige o token no cabeçalho `X-Monitor-Token`:

```http
POST /api/events/clear HTTP/1.1
X-Monitor-Token: your-secret-token
```

Omitir o token, ou enviar um valor errado, devolve `401 Unauthorized`.

---

### `POST /api/events/cleanup-rotated`

Apaga ficheiros de eventos arquivados (rodados) com mais de um determinado número de dias.

| Campo do corpo | Padrão | Descrição |
|---|---|---|
| `max_age_days` | `7` | Apaga arquivos rodados com mais de este número de dias. Mínimo `1`. |

```json
{ "max_age_days": 14 }
```

Devolve `{ "ok": true, "deleted": 3 }`, onde `deleted` é o número de ficheiros removidos.

---

## Server-Sent Events

### `GET /api/events/stream`

Transmite novos registos JSONL completos em mensagens `data:`. Os heartbeats são eventos chamados `ping` com um timestamp, não strings literais `data: ping`:

```text
data: {"type":"refresh","ts":1714310561,"instance":"a1b2c3d4","payload":{"key":"users:42","driver":"FileStore","duration_ms":0,"success":true}}

event: ping
data: {"ts":1714310563}

```

A ligação dura `CACHEER_MONITOR_STREAM_TIMEOUT` segundos (padrão `30`). O `EventSource` volta a ligar automaticamente após timeout ou falha de rede. O dashboard obtém um snapshot ao ligar e agrupa rajadas de eventos em atualizações; os pings de heartbeat não pedem uma. A rotação/truncagem reinicia a leitura no novo ficheiro, e linhas incompletas aguardam conclusão.

O polling periódico continua disponível no intervalo selecionado. **Manual** para o polling, mas não desativa as atualizações SSE. As regras de saúde do navegador são avaliadas após atualizações bem-sucedidas do snapshot; `/api/health` continua a verificar a disponibilidade do servidor e não reporta os resultados das regras.
