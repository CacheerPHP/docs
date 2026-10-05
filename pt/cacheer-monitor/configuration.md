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
| `CACHEER_MONITOR_AUTO_REGISTER` | `true` | Defina como `false` para impedir que a ponte de autoload registe um listener, permitindo registar o seu |
| `CACHEER_MONITOR_STREAM_TIMEOUT` | `30` | Quanto tempo (segundos) a ligação SSE `/api/events/stream` fica aberta antes de o cliente ter de se religar. |
| `CACHEER_MONITOR_WORKERS` | `4` | Processos de trabalho do servidor local em Unix. Uma ligação SSE ocupa um processo até fechar. |
| `CACHEER_MONITOR_PREVIEW_BYTES` | `2048` | Tamanho máximo (bytes) da pré-visualização serializada escrita em cada evento. Valores maiores são truncados. |
| `CACHEER_MONITOR_REDACT_KEYS` | — | Lista separada por vírgulas de campos adicionais a ocultar nas pré-visualizações. Campos ocultados por omissão: `password`, `passwd`, `pwd`, `secret`, `token`, `access_token`, `refresh_token`, `authorization`, `api_key`, `apikey`, `private_key`, `client_secret`, `cookie`, `session`. |

## Frequência de Atualização

- O intervalo de atualização da UI é de **5 s** por omissão (ajustável no seletor do cabeçalho).
- A atualização manual está disponível no botão **Refresh** do dashboard.
- O SSE atualiza o dashboard quando chegam novos registos e sincroniza após voltar a ligar. Os pings de heartbeat mantêm o fluxo ativo; não pedem uma atualização do snapshot.
- **Manual** para o polling periódico. O SSE pode continuar a atualizar o dashboard quando chegam eventos ou a ligação é restabelecida.
- Para ficheiros grandes, considere limpar via `POST /api/events/clear` depois de arquivar.

## Captura de Valores

Com `CACHEER_MONITOR_CAPTURE_VALUES=true`, eventos que transportam um valor podem incluir `value_preview` com uma pré-visualização em JSON (até `CACHEER_MONITOR_PREVIEW_BYTES` bytes). Este campo alimenta a pré-visualização do Key Inspector; os sinais de ciclo de vida não transportam um valor em cache.

Campos sensíveis são ocultados automaticamente nas pré-visualizações. Amplie a lista com `CACHEER_MONITOR_REDACT_KEYS`:

```env
CACHEER_MONITOR_CAPTURE_VALUES=true
CACHEER_MONITOR_REDACT_KEYS=ssn,credit_card,dob
```

> **Nota:** A captura de valores guarda dados no log JSONL. Evite ativá-la em produção ou em ficheiros com retenção longa.

## Regras de Saúde do Dashboard

Abra **Health rules**, abaixo de **Cache lifecycle**. Estas definições ficam guardadas no navegador, separadas das variáveis de ambiente. Aplicam-se às atualizações bem-sucedidas enquanto a página está aberta, usando o intervalo temporal e namespace atuais. A pesquisa de chaves e o filtro de operação não afetam as regras.

| Definição | Padrão | Comportamento |
|---|---|---|
| **Low hit-rate warning** | Ativo | Avisa quando a taxa de acerto está abaixo de **Alert below**, no cartão de eficiência. |
| **Alert below** | `50%` | Limiar de taxa de acerto; só as consultas de cache contam para o mínimo de amostras. |
| **Error-count warning** | Desativado | Quando ativo, avisa ao atingir ou ultrapassar o número configurado de erros no intervalo. |
| Limiar de erros | `5` | Contagem de erros, não percentagem; os eventos registados contam para o mínimo de amostras. |
| **p95 latency warning** | Desativado | Quando ativo, avisa quando a latência p95 das operações de cache excede o limiar. |
| Limiar de latência p95 | `10 ms` | Só as operações com duração contam para o mínimo de amostras. |
| **Minimum samples** | `10` | Consultas para a taxa de acerto, eventos registados para erros e operações com duração para latência. |
| **Condition holds for** | `30 segundos` | A condição deve continuar a ser observada entre atualizações durante este período antes de avisar. `0` elimina o período de espera. |
| **Snooze / repeat cooldown** | `60 segundos` | Suprime avisos adiados com snooze e repetições após recuperação. |

Os avisos ativos permanecem visíveis até a condição recuperar ou selecionar **Snooze**. O cooldown não oculta automaticamente um aviso ativo. Desative uma regra na respetiva caixa de seleção.

Alterar definições, intervalo temporal ou namespace reinicia a avaliação. Uma atualização falhada também a reinicia. Recarregar a página inicia uma nova avaliação e limpa os snoozes, mantendo as definições guardadas. Não há serviço de alertas em segundo plano nem notificações com o dashboard fechado.

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
  "instance": "a1b2c3d4",
  "payload": {
    "key": "users:42",
    "driver": "FileStore",
    "success": true,
    "size_bytes": 2048,
    "duration_ms": 1.9,
    "value_preview": "{\"id\":42,\"name\":\"Alice\"}"
  }
}
```

`value_preview`, `value_type`, `size_bytes`, `count` e `error` dependem do evento e da captura ativa. Os registos atuais da v6 usam os tipos `hit`, `miss`, `put`, `clear`, `flush`, `prune`, `error`, `promotion`, `stale_served`, `refresh` e `lock_contended`. Os sinais de ciclo de vida atuais não têm duração medida, mesmo que o registo contenha `duration_ms: 0`; o Monitor exclui-os das estatísticas de latência.

O tipo de evento atual da v6 não transporta metadados de namespace ou TTL. Eventos antigos ou personalizados podem incluir `namespace` e `ttl`. Um namespace explicitamente vazio é o escopo padrão; um campo em falta não foi reportado. Nas escritas, `ttl: null` explícito significa duração ilimitada, enquanto a ausência de `ttl` significa desconhecido. O dashboard explica metadados em falta em vez de inventar valores.
