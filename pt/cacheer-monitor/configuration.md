# Configuração & Ambiente

O monitor privilegia a convenção, mas alguns ajustes mantêm-no flexível durante o desenvolvimento.

## Resolução do Ficheiro de Eventos

O monitor resolve o ficheiro de eventos por esta ordem:

1. **Ambiente do SO** — `CACHEER_MONITOR_EVENTS` definido na shell ou no ambiente do processo (origem: `env`)
2. **Ficheiro `.env`** — lido a partir da raiz do projeto via `src/Support/Env.php` (origem: `dotenv`)
3. **Fallback** — `sys_get_temp_dir()/cacheer-monitor.jsonl` (origem: `default`)

Caminhos relativos são resolvidos em relação à raiz do projeto consumidor, nunca ao diretório do pacote do monitor dentro de `vendor/`.

## Variáveis de Ambiente

Todas as variáveis podem ser definidas num ficheiro `.env` na raiz do projeto ou como variáveis de ambiente reais do SO. Use o `.env.example` do pacote do monitor como ponto de partida:

```bash
cp vendor/cacheerphp/monitor/.env.example .env
```

| Variável | Padrão | Descrição |
|---|---|---|
| `CACHEER_MONITOR_EVENTS` | *(diretório temporário)* | Caminho absoluto, ou relativo à raiz do projeto, para o ficheiro de eventos JSONL. |
| `CACHEER_MONITOR_TOKEN` | — | Quando definido, as ações destrutivas da API (`clear`, `cleanup-rotated`) exigem este valor no cabeçalho `X-Monitor-Token`. |
| `CACHEER_MONITOR_CAPTURE_VALUES` | `false` | Quando `true`, a camada de instrumentação grava uma pré-visualização dos valores em cache no payload de cada evento. Ver [Captura de Valores](#captura-de-valores) abaixo. |
| `CACHEER_MONITOR_STREAM_TIMEOUT` | `30` | Quanto tempo (segundos) a ligação SSE `/api/events/stream` fica aberta antes de o cliente ter de se religar. |
| `CACHEER_MONITOR_PREVIEW_BYTES` | `2048` | Tamanho máximo (bytes) da pré-visualização serializada escrita em cada evento. Valores maiores são truncados. |
| `CACHEER_MONITOR_REDACT_KEYS` | — | Lista separada por vírgulas de campos adicionais a ocultar nas pré-visualizações. Campos ocultados por omissão: `password`, `passwd`, `pwd`, `secret`, `token`, `access_token`, `refresh_token`, `authorization`, `api_key`, `apikey`, `private_key`, `client_secret`, `cookie`, `session`. |

## Frequência de Atualização

- O intervalo de atualização da UI é de **5 s** por omissão (ajustável no seletor do cabeçalho).
- A atualização manual está disponível no botão **Refresh** do dashboard.
- O fluxo de eventos usa pings SSE como fallback quando suportado.
- Para ficheiros grandes, considere limpar via `POST /api/events/clear` depois de arquivar.

## Captura de Valores

Com `CACHEER_MONITOR_CAPTURE_VALUES=true`, cada evento de cache inclui um campo `value_preview` com um snapshot JSON do valor em cache (até `CACHEER_MONITOR_PREVIEW_BYTES` bytes). É isto que alimenta a pré-visualização do painel Key Inspector.

Campos sensíveis são ocultados automaticamente nas pré-visualizações. Amplie a lista com `CACHEER_MONITOR_REDACT_KEYS`:

```env
CACHEER_MONITOR_CAPTURE_VALUES=true
CACHEER_MONITOR_REDACT_KEYS=ssn,credit_card,dob
```

> **Nota:** A captura de valores guarda dados no log JSONL. Evite ativá-la em produção ou em ficheiros com retenção longa.

## Proteção da API por Token

Defina `CACHEER_MONITOR_TOKEN` para proteger os endpoints destrutivos:

```env
CACHEER_MONITOR_TOKEN=my-local-secret
```

Passe o token como cabeçalho HTTP ao chamar `POST /api/events/clear` ou `POST /api/events/cleanup-rotated`:

```bash
curl -X POST http://127.0.0.1:9966/api/events/clear \
  -H "X-Monitor-Token: my-local-secret"
```

## Formato do Payload de Evento

Cada linha do ficheiro JSONL é um único evento do Cacheer:

```json
{
  "type": "hit",
  "ts": 1714310556,
  "payload": {
    "key": "users:42",
    "namespace": "default",
    "driver": "FileCacheStore",
    "size_bytes": 2048,
    "duration_ms": 1.9,
    "ttl": 86400,
    "value_preview": "{\"id\":42,\"name\":\"Alice\"}"
  }
}
```

Campos adicionais (`ttl`, `value_preview`, `value_type`, `size_bytes`, `exists`, `count`) aparecem consoante o tipo de evento e as funcionalidades ativas.
