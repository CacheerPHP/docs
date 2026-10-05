# Resolução de Problemas

Quando o dashboard parece vazio, verifique isto primeiro:

| Sintoma | O que verificar |
|---|---|
| Nenhum evento visível | Confirme o caminho do ficheiro de eventos via `GET /api/config` e garanta que o ficheiro existe |
| Aviso de `.env` não encontrado no arranque | Crie um `.env` na raiz do projeto: `cp vendor/cacheerphp/monitor/.env.example .env` — o monitor funciona sem ele, mas os eventos vão para o diretório temporário do sistema |
| Erro de permissões | O processo PHP tem de conseguir ler **e** escrever no ficheiro JSONL |
| UI lenta | Rode o log com `POST /api/events/clear` se o ficheiro tiver crescido muito |
| O servidor não arranca | Requer PHP 8.3 ou mais recente na integração v6. Execute `vendor/bin/cacheer-monitor doctor` para verificar a compatibilidade. |
| Assets desatualizados | Faça hard-refresh no navegador depois de atualizar os assets do monitor |
| O driver aparece sempre como `unknown` | O listener v6 lê o driver do campo `store` do evento tipado. Execute `vendor/bin/cacheer-monitor doctor`, confirme que o listener de telemetria está ativo e que a operação emite um evento instrumentado. Não é necessária reflexão nem um accessor `getCacheStore()`. |
| TTL não aparece no Key Inspector | A ponte atual de telemetria v6 não emite metadados de TTL. Configure a expiração com `set($key, $value, ttl: ...)` ou `defaultTtl(...)` no builder; a ausência de TTL no dashboard não significa que a entrada nunca expira. |
| Pré-visualização de valores em falta no Key Inspector | Ative a captura de valores: defina `CACHEER_MONITOR_CAPTURE_VALUES=true` no `.env`. Use o ícone de atualização do inspetor (**Refresh live value**) para pedir uma leitura ao vivo. O processo do Monitor deve conseguir resolver a store. |
| `401 Unauthorized` em clear / cleanup | `CACHEER_MONITOR_TOKEN` está definido. Inclua o token no cabeçalho `X-Monitor-Token: <token>`. |
| Aviso de saúde não aparece | Abra **Health rules** e ative a respetiva caixa de seleção. Verifique o limiar, **Minimum samples** e **Condition holds for**. Por omissão, exige 10 amostras e uma condição observada durante 30 segundos; as regras de erros/latência começam desativadas. Condições adiadas com snooze ou recuperadas também respeitam o cooldown. Sem consultas, a taxa de acerto mostra `—`, não uma falha. |
| Aviso desaparece após recarregar ou alterar filtros | As definições guardadas permanecem, mas alterar definições, intervalo temporal ou namespace, recarregar ou falhar uma atualização inicia uma nova avaliação. Os snoozes não sobrevivem ao recarregamento. Os filtros de chave/tipo não alteram as regras. |
| Filtro de namespace indisponível | Os eventos atuais da v6 não reportam metadados de namespace. O dashboard indica a indisponibilidade em vez de tratar toda a atividade como escopo padrão. Registos antigos/personalizados podem fornecer um namespace explícito. Um filtro ativo continua editável mesmo sem correspondências. |
| Problem keys está vazio | O ranking selecionado só inclui chaves com misses, erros, hits ou operações com duração registados. Limpe a pesquisa, alargue o intervalo ou selecione **Most hits**. Não haver erros registados é um resultado vazio válido. |
| Contagens do ciclo de vida a zero | Contam sinais emitidos, não todas as consultas. Valores antigos servidos, atualizações, promoções entre camadas e timeouts de locks de single-flight só aparecem quando esses comportamentos ocorrem e são instrumentados. |
| Limite do fluxo não altera as métricas de resumo | É esperado. As métricas e gráficos usam todos os registos correspondentes do log atual; o limite só altera os eventos apresentados. Os filtros de chave/tipo também mantêm as métricas de resumo. |
| Histórico antigo desaparece após rotação do log | O dashboard só lê o log atual, incluindo em **All time** e na inspeção de chaves. Exporte ou retenha os ficheiros arquivados separadamente quando precisar de mais histórico. |
| O fluxo indica que está a voltar a ligar | As ligações SSE fecham após `CACHEER_MONITOR_STREAM_TIMEOUT` (30 segundos por omissão) e voltam a ligar automaticamente. **Connected** indica separadamente a ligação à API/snapshot. O polling continua no intervalo escolhido; **Manual** para o polling mas permite atualizações SSE. |
| Filtro temporal não devolve eventos | Os parâmetros `from`/`until` são timestamps Unix em segundos (float). Se os eventos foram capturados com um relógio diferente, alargue o intervalo ou volte a **All** para confirmar que existem eventos. |
