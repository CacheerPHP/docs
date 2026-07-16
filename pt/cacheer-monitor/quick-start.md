# Início Rápido

Ponha a telemetria a chegar ao dashboard em menos de um minuto.

## Instalar

```bash
composer require cacheerphp/monitor
```

**A integração é só isto.** O pacote regista-se automaticamente através do `autoload.files` do Composer — assim que o `vendor/autoload.php` é carregado, todas as operações de cache de todas as instâncias `Cacheer` passam a ser instrumentadas automaticamente. Não é preciso alterar código.

## Iniciar o Dashboard

Num terminal separado, a partir da raiz do projeto:

```bash
vendor/bin/cacheer-monitor serve --port=9966
```

Abra [http://127.0.0.1:9966](http://127.0.0.1:9966) no navegador e execute a aplicação normalmente — o dashboard atualiza-se em tempo real.

> **Nota:** Se o projeto não tiver um ficheiro `.env`, o monitor funciona na mesma — os eventos serão guardados no diretório temporário do sistema. Para um caminho persistente e previsível, crie um `.env` na raiz do projeto. Pode usar o do pacote CacheerPHP como ponto de partida:
>
> ```bash
> cp vendor/silviooosilva/cacheer-php/.env.example .env
> ```
>
> Depois adicione a linha seguinte para definir o caminho do ficheiro de eventos:
>
> ```env
> CACHEER_MONITOR_EVENTS=/o/seu/caminho/cacheer-monitor.jsonl
> ```

---

## Caminho Personalizado do Ficheiro de Eventos

Os eventos são escritos no caminho resolvido por esta ordem:

1. Variável de ambiente `CACHEER_MONITOR_EVENTS`
2. Ficheiro `.env` na raiz do projeto
3. Diretório temporário do sistema (`sys_get_temp_dir() . '/cacheer-monitor.jsonl'`)

Para substituir, remova o listener registado automaticamente e adicione o seu depois do `vendor/autoload.php`:

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Cacheer\Monitor\CacheerMonitorListener;
use Cacheer\Monitor\Reporter\JsonlReporter;

Cacheer::removeListeners();
Cacheer::addListener(new CacheerMonitorListener(
    new JsonlReporter('/var/log/myapp/cacheer-events.jsonl')
));
```

Inicie o servidor a apontar para o mesmo caminho:

```bash
CACHEER_MONITOR_EVENTS=/var/log/myapp/cacheer-events.jsonl \
  vendor/bin/cacheer-monitor serve --port=9966
```

---

## Múltiplos Listeners

Pode registar mais do que um listener — útil para enviar eventos para backends diferentes em simultâneo:

```php
Cacheer::addListener(new CacheerMonitorListener(new JsonlReporter()));
Cacheer::addListener(new MyCustomAlerter());
```

Para remover todos os listeners:

```php
Cacheer::removeListeners();
```

---

## Legado: Registo Manual

Antes da v1.0.0, a abordagem recomendada era envolver a instância `Cacheer` manualmente ou chamar `addListener` à mão. Ambas continuam a funcionar, mas já não são necessárias na maioria dos projetos:

```php
// Listener manual (continua válido para configuração personalizada):
Cacheer::addListener(new CacheerMonitorListener(new JsonlReporter()));

// Abordagem antiga com wrapper (funciona, mas exige alterações de código):
$instrumented = InstrumentedCacheer::wrap($cacheer, new JsonlReporter());
```
