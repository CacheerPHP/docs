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
| Pré-visualização de valores em falta no Key Inspector | Ative a captura de valores: defina `CACHEER_MONITOR_CAPTURE_VALUES=true` no `.env`. Depois use **Refresh Live** no inspetor para forçar uma leitura ao vivo do cache. |
| `401 Unauthorized` em clear / cleanup | `CACHEER_MONITOR_TOKEN` está definido. Inclua o token no cabeçalho `X-Monitor-Token: <token>`. |
| Banner de alerta da taxa de acerto não aparece | Introduza um valor diferente de zero no campo **Alert Threshold %** e confirme que a taxa de acerto atual está de facto abaixo dele. O banner só ativa quando ambas as condições se verificam. Se ainda não houver eventos capturados, a taxa pode aparecer como `0` ou `100` consoante a janela de filtro. |
| Filtro temporal não devolve eventos | Os parâmetros `from`/`until` são timestamps Unix em segundos (float). Se os eventos foram capturados com um relógio diferente, alargue o intervalo ou volte a **All** para confirmar que existem eventos. |
