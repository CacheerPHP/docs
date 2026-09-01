# Início Rápido

Ponha a telemetria a chegar ao dashboard em menos de um minuto.

## Instalar

```bash
composer require cacheerphp/monitor
```

**A integração é só isto.** O pacote regista-se automaticamente através do `autoload.files` do Composer — assim que o `vendor/autoload.php` é carregado, regista um listener no tap global de telemetria do CacheerPHP 6 e todos os caches passam a reportar automaticamente. Não é preciso alterar código.

Isto cobre todas as formas de construir um cache: os construtores nomeados, o `Cacheer::build()`, um simples `new Cacheer($store)` e os decorators `tiered()` / `resilient()`. Vistas com escopo ou com policy herdam-no, e as operações de capacidade (`increment`, `decrement`, `touch`, `tag`, `flushTag`) também reportam.

> Requer **CacheerPHP 6**. Enquanto a 6.0 for uma pré-lançamento, a dependência usa `^6.0@RC`, que fixa na etiqueta `6.0.0-RC1`; veja [quando não aparece nada](#quando-não-aparece-nada).

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

A substituição mais simples não precisa de código:

```bash
CACHEER_MONITOR_EVENTS=/var/log/myapp/cacheer-events.jsonl
```

Inicie o servidor apontando para o mesmo caminho:

```bash
CACHEER_MONITOR_EVENTS=/var/log/myapp/cacheer-events.jsonl \
  vendor/bin/cacheer-monitor serve --port=9966
```

---

## Registar um listener manualmente

Desligue primeiro a ponte, ou ficará com dois listeners a escrever:

```env
CACHEER_MONITOR_AUTO_REGISTER=false
```

```php
use Cacheer\Monitor\CacheerMonitorListener;
use Cacheer\Monitor\Reporter\JsonlReporter;
use Silviooosilva\CacheerPhp\Observability\Telemetry;

Telemetry::listen(
    (new CacheerMonitorListener(new JsonlReporter('/var/log/myapp/cacheer-events.jsonl')))->dispatch(...)
);
```

## Vários listeners

O tap distribui os eventos, por isso registe quantos quiser — útil para enviar
eventos para vários backends ao mesmo tempo:

```php
Telemetry::listen((new CacheerMonitorListener(new JsonlReporter()))->dispatch(...));
Telemetry::listen($meuAlerta->handle(...));
```

Um listener que lance uma exceção nunca quebra uma operação de cache; o tap
absorve a falha. Para remover todos os listeners (incluindo o automático):

```php
Telemetry::reset();
```

## Instrumentar apenas um cache

A ponte é deliberadamente tudo-ou-nada. Para instrumentar um único cache,
desligue-a e use o construtor `instrumented()` do próprio CacheerPHP:

```php
use Silviooosilva\CacheerPhp\Cacheer;
use Silviooosilva\CacheerPhp\Observability\EventBus;

$events = new EventBus();
$events->listen((new CacheerMonitorListener(new JsonlReporter()))->dispatch(...));

$cache = Cacheer::instrumented($store, $events);   // só este reporta
```

---

## Quando não aparece nada

A ponte corre no autoload em cada pedido, por isso nunca pode avisar nem lançar
exceções — o silêncio é o seu único modo de falha seguro. O `doctor` é onde esse
silêncio é explicado:

```bash
vendor/bin/cacheer-monitor doctor
```

```
  [ok]  CacheerPHP telemetry tap Observability\Telemetry found
  [ok]  Autoload bridge       active
  [ok]  Listener registered   caches will report
  [ok]  Events file           /tmp/cacheer-monitor.jsonl
```

Sai com código diferente de zero quando uma verificação falha, por isso funciona
em CI. As duas causas habituais:

- **O ficheiro de eventos.** Sem `CACHEER_MONITOR_EVENTS` definido, o caminho por
  omissão fica no diretório temporário do sistema — logo "não reporta nada" é
  muitas vezes "o dashboard está a ler outro ficheiro".
- **A versão do CacheerPHP.** O monitor liga-se ao tap de telemetria do
  CacheerPHP 6, que não existe na v4/v5. Enquanto a 6.0 for uma pré-lançamento, a
  restrição é `^6.0@RC` — um `^6.0` simples falha silenciosamente o
  `minimum-stability: stable` e deixa instalada uma versão antiga sem tap onde
  ligar.
