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
| `CACHEER_MONITOR_ALLOW_ORIGIN` | Não definido | Ativa acesso à API a partir de outra origem explícita. O dashboard incluído usa a mesma origem. |
| `CACHEER_MONITOR_CACHE_PATH` | Não definido | Diretório de cache da aplicação para pré-visualizações FileStore ao vivo. |
| `CACHEER_MONITOR_CACHE_TABLE` | `cacheer_store` | Tabela da aplicação para pré-visualizações DatabaseStore ao vivo. |
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

Os eventos do CacheerPHP 6.1+ transportam namespace, TTL e operação; os da versão 6.0 não. Eventos antigos ou personalizados também podem incluir `namespace` e `ttl`. Um namespace explicitamente vazio é o escopo padrão; um campo em falta não foi reportado. Nas escritas, `ttl: null` explícito significa duração ilimitada, enquanto a ausência de `ttl` significa desconhecido. O dashboard explica metadados em falta em vez de inventar valores.

## Pré-visualizações ao vivo

Os eventos capturados são a fonte principal das pré-visualizações. Com captura
ativa, o inspector também pode ler valores de FileStore, DatabaseStore ou
RedisStore usando as configurações partilhadas da aplicação: `DB_*`, `REDIS_*`
e os caminhos/tabelas acima. Estas leituras não emitem telemetria.

As leituras ao vivo usam o pipeline e o codificador de chaves predefinidos.
Para codecs, compressão ou criptografia personalizados, ou stores locais ao
processo como ArrayStore, ative a captura de valores na aplicação. O inspector
explica leituras indisponíveis e mantém a última pré-visualização capturada
quando a atualização falha. Um nome de apresentação com escopo só é resolvido
ao vivo se identificar um único valor; nomes ambíguos usam a captura.

## Escrita em lotes e rotação

O `JsonlReporter` acumula linhas JSONL e escreve cada lote sob o `.lock`
partilhado do ficheiro de eventos. Cada instância do reporter atribui um ID
`instance` para diagnósticos entre processos. A rotação ocorre aos **10 MiB**.

Um lote é escrito aos **256 eventos**, no primeiro evento pelo menos **um
segundo** depois da escrita anterior, numa chamada a `$reporter->flush()` e no
fim do processo, incluindo erros fatais de PHP. Com PHP-FPM, os eventos aparecem
quando o pedido termina. Um worker inativo só escreve o último lote na próxima
operação de cache ou ao sair; chame `flush()` no fim de cada pedido/job para
escrever mais cedo. Um processo terminado sem shutdown, como por `SIGKILL`, pode
perder o lote em memória.

Os processos intercalam lotes no ficheiro. O fluxo do snapshot e o histórico das
chaves são ordenados pelo timestamp. O snapshot mantém os eventos mais recentes
antes de aplicar o limite; a ordem no ficheiro desempata timestamps iguais.

Uma medição local do reporter com lotes registou cerca de **2,5 microssegundos
por `get()` com o Monitor instalado**. O resultado depende da carga e da máquina;
não é uma garantia de latência.

## Acesso local e cabeçalhos

O servidor escuta em `127.0.0.1` por omissão. O token opcional protege ações
destrutivas, não autentica leituras nem torna a publicação segura. Adicione
controlo de acesso separado antes de expor o dashboard à rede.

As respostas normais da API usam `Cache-Control: no-store`; o stream SSE usa
`no-cache`. O acesso entre origens está desligado por omissão. Defina
`CACHEER_MONITOR_ALLOW_ORIGIN` apenas quando outra origem precisar de acesso;
CORS não autentica utilizadores. A ocultação de campos sensíveis é uma proteção
parcial: reveja dados capturados antes de partilhar logs ou exportações.

## Operações e eventos por versão do núcleo

Os mapeamentos base são `Hit` → `hit`, `Miss` → `miss`, `Write` → `put`,
`Delete` → `clear`, `Clear` → `flush`, `Prune` → `prune` e `Failure` → `error`.
Apagar uma chave e limpar a store são operações diferentes. Os eventos do ciclo
de vida são `promotion`, `stale_served`, `refresh` e `lock_contended`.

Com CacheerPHP 6.1+, a operação reportada distingue `touch` → `renew`, `tag` →
`tag`, `increment` → `increment` e `clearTag` → `flush_tag`. Execuções de loaders
usam `computed`; o failover usa `fallback_served`, `circuit_opened`,
`circuit_closed` e `reconciled`. Estes relatórios não entram na latência das
operações de cache. Sem metadados de operação, aplica-se o mapeamento base.
