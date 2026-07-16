# Referência da CLI

Os comandos auxiliares vivem em `bin/cacheer-monitor`.

## `serve`

Inicia a API HTTP e os assets da SPA usando o servidor embutido do PHP.

| Flag | Padrão | Descrição |
|---|---|---|
| `--host` | `127.0.0.1` | Endereço de bind |
| `--port` | `9966` | Porta de escuta |
| `--quiet` | — | Suprime o log de pedidos |

```bash
php bin/cacheer-monitor serve --host=0.0.0.0 --port=9000 --quiet
```

## Scripts & Reporters

Execute as demos incluídas para exercitar os adaptadores do Cacheer e gerar tráfego realista.

| Comando | Descrição |
|---|---|
| `php Tests/scenarios.php` | Mistura de tráfego base |
| `php Tests/scenarios_advanced.php` | Carga em rajadas, flush por tags, eventos de erro |
| `php Tests/metrics_test.php` | Verificação rápida de regressão da agregação de métricas |
