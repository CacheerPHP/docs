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

## `doctor`

Verifica a ponte de autoload de ponta a ponta e explica por que está inativa
quando é o caso.

```bash
vendor/bin/cacheer-monitor doctor
```

```
  [ok]  CacheerPHP telemetry tap Observability\Telemetry found
  [ok]  Autoload bridge       active
  [ok]  Listener registered   caches will report
  [ok]  Events file           /tmp/cacheer-monitor.jsonl
```

Confirma que existe um CacheerPHP com o tap de telemetria instalado, que o
Composer executou o bootstrap, que há um listener registado e que o ficheiro de
eventos é gravável. Sai com código diferente de zero quando uma verificação
falha, por isso funciona em CI.

A ponte corre no autoload em cada pedido e por isso nunca pode avisar nem lançar
exceções — este comando é onde o seu silêncio é explicado.

## Scripts & Reporters

Execute as demos incluídas para exercitar os adaptadores do Cacheer e gerar tráfego realista.

| Comando | Descrição |
|---|---|
| `php Tests/scenarios.php` | Mistura de tráfego base |
| `php Tests/scenarios_advanced.php` | Carga em rajadas, flush por tags, eventos de erro |
| `php Tests/metrics_test.php` | Verificação rápida de regressão da agregação de métricas |
