# Exemplo 16 — Integração com o Monitor *(v5.0.0)*

Adicione telemetria em tempo real a um projeto CacheerPHP existente com um único comando — sem alterações de código.

## Instalar

```bash
composer require cacheerphp/monitor
```

O pacote regista-se automaticamente através do `autoload.files` do Composer. A partir daqui, todas as operações de cache são instrumentadas automaticamente.

## Código Existente — Sem Alterações

```php
require 'vendor/autoload.php'; // o bootstrap.php corre aqui — o monitor fica ativo

use Silviooosilva\CacheerPhp\Cacheer;

$cacheer = new Cacheer(['cacheDir' => __DIR__ . '/cache']);
$cacheer->setDriver()->useFileDriver();

// Todas estas chamadas emitem eventos de telemetria automaticamente:
$cacheer->putCache('user:1', ['name' => 'Alice']);
$cacheer->getCache('user:1');           // → evento 'hit'
$cacheer->getCache('user:99');          // → evento 'miss'
$cacheer->increment('page_views');
$cacheer->clearCache('user:1');
$cacheer->flushCache();                 // → evento 'flush'

// A fachada estática também dispara eventos:
Cacheer::putCache('config:locale', 'pt_PT');
Cacheer::getCache('config:locale');     // → evento 'hit'
```

## Ver o Dashboard

```bash
vendor/bin/cacheer-monitor serve --port=9966
# → http://127.0.0.1:9966
```

## Caminho de Ficheiro Personalizado

Se precisar de um ficheiro de eventos personalizado, substitua depois do autoload:

```php
use Cacheer\Monitor\CacheerMonitorListener;
use Cacheer\Monitor\Reporter\JsonlReporter;

Cacheer::removeListeners();
Cacheer::addListener(new CacheerMonitorListener(
    new JsonlReporter('/var/log/myapp/events.jsonl')
));
```

## Eventos Emitidos

| Operação | Tipo de evento |
|---|---|
| `getCache` (encontrado) | `hit` |
| `getCache` (não encontrado) | `miss` |
| `putCache` | `put` |
| `putMany` | `put_many` |
| `clearCache` | `clear` |
| `flushCache` | `flush` |
| `increment` / `decrement` | `increment` / `decrement` |
| `add` | `add` |
| `remember` / `rememberForever` | `remember` / `remember_forever` |
| `forever` | `put_forever` |
| `renewCache` | `renew` |
| `tag` / `flushTag` | `tag` / `flush_tag` |
| `has` | `has` |
| `getAndForget` | `get_and_forget` |

Cada evento inclui: `key`, `namespace` (quando não vazio), `driver`, `duration_ms` e `success`.

## Ver Também

- [Cacheer Monitor — Início Rápido](../cacheer-monitor/quick-start.md)
- [Cacheer Monitor — API REST](../cacheer-monitor/api.md)
- [Exemplo 15 — Estatísticas e Gestão de Instâncias](./exemplo-15-estatisticas-instancia.md)
