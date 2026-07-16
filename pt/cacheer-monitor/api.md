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

Resume todos os eventos em contadores, taxa de acerto e percentis de latência.

| Parâmetro | Padrão | Descrição |
|---|---|---|
| `namespace` | — | Restringe as métricas a um único namespace. |
| `limit` | `1000` | Número máximo de eventos a agregar. |
| `from` | — | Timestamp Unix (float). Inclui apenas eventos a partir deste instante. |
| `until` | — | Timestamp Unix (float). Inclui apenas eventos até este instante. |

```json
{
  "hits": 1200,
  "misses": 45,
  "hit_rate": 0.963,
  "latency": { "avg_ms": 3.1, "p95_ms": 7.8, "p99_ms": 15.2 },
  "drivers": { "redis": 1090, "array": 155 },
  "namespaces": { "default": 800, "critical": 290 },
  "top_keys": { "users:42": 120, "orders:recent": 56 },
  "ttl_distribution": {
    "forever": 10, "gt_1day": 40, "gt_1hour": 120,
    "gt_5min": 80, "gt_1min": 30, "lte_1min": 5
  }
}
```

---

### `GET /api/events`

Devolve os eventos mais recentes primeiro. Filtrável por namespace, tipo ou fragmento de chave.

| Parâmetro | Padrão | Descrição |
|---|---|---|
| `limit` | `200` | Número máximo de eventos devolvidos. |
| `namespace` | — | Restringe a um único namespace. |
| `from` | — | Timestamp Unix (float). Inclui apenas eventos a partir deste instante. |
| `until` | — | Timestamp Unix (float). Inclui apenas eventos até este instante. |

```json
{
  "type": "put",
  "ts": 1714310556,
  "payload": {
    "key": "users:42",
    "namespace": "default",
    "driver": "FileCacheStore",
    "ttl": 86400,
    "duration_ms": 5.4
  }
}
```

---

### `GET /api/keys/inspect`

Devolve um resumo por chave e os eventos mais recentes dessa chave. Alimenta o painel Key Inspector do dashboard.

| Parâmetro | Padrão | Descrição |
|---|---|---|
| `key` | **obrigatório** | A chave exata a inspecionar. |
| `namespace` | — | Restringe a um namespace específico. |
| `limit` | `100` | Número máximo de eventos da chave devolvidos. |
| `live` | `false` | Quando `true` e a captura de valores está ativa, força uma leitura ao vivo do cache para preencher a pré-visualização. |

```json
{
  "summary": {
    "key": "users:42",
    "hits": 18,
    "misses": 2,
    "puts": 3,
    "hit_rate": 0.9,
    "last_put_at": 1714310556,
    "last_hit_at": 1714310900,
    "last_miss_at": 1714309100,
    "last_ttl": 86400,
    "last_size_bytes": 512,
    "last_value_type": "array",
    "last_value_preview": "{\"id\":42,\"name\":\"Alice\"}",
    "capture_values_enabled": true,
    "namespaces": { "default": 20 },
    "drivers": { "FileCacheStore": 20 }
  },
  "events": [ ... ]
}
```

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

Fornece um heartbeat leve usado pela SPA para atualizar a lista de eventos à medida que novas linhas chegam ao ficheiro JSONL. As mensagens são payloads simples `data: ping`.

A ligação fica aberta durante `CACHEER_MONITOR_STREAM_TIMEOUT` segundos (padrão `30`). Quando `EventSource` não está disponível, o dashboard recorre a polling com o intervalo de atualização configurado.
