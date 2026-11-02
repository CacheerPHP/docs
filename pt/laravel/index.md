# CacheerPHP para Laravel

O CacheerPHP para Laravel (`cacheerphp/laravel`) coloca o CacheerPHP por baixo do cache do Laravel. Depois de o instalar, as stores que já tem passam a correr através do CacheerPHP com os mesmos resultados e os mesmos dados: o código e o `config/cache.php` ficam como estão, e o [Cacheer Monitor](../cacheer-monitor/index.md) pode mostrar o seu tráfego. Para o armazenamento próprio do CacheerPHP, com failover, cache em camadas e criptografia em repouso, adicione uma store que use o driver `cacheer`.

Requisitos: PHP 8.3+, Laravel 12 ou 13 e CacheerPHP 6.1.1 ou posterior, que o pacote instala.

## Escolher como usar o CacheerPHP no Laravel

| Pretende… | Use |
| --- | --- |
| As stores de cache que já tem, observadas no Cacheer Monitor, sem mais alterações | **O encaminhamento**, ativo assim que o pacote é instalado |
| Failover, cache em camadas ou criptografia para o cache do Laravel | **O driver `cacheer`**, uma store sobre o armazenamento próprio do CacheerPHP |
| Um cache CacheerPHP separado para os seus serviços | [Um binding no container](../guias/laravel.md) |

Pode combiná-los: o encaminhamento funciona ao lado das stores `cacheer`, e o driver também regista o contrato `Cache` do CacheerPHP, para que os serviços o possam injetar e usar funcionalidades que `Cache::` não expressa, como a invalidação por tags.

## Stores encaminhadas

Cada store que o Laravel constrói a partir de `config/cache.php`, como `redis`, `database` ou `file`, passa a correr através do CacheerPHP. As leituras e escritas passam pelo CacheerPHP no formato da própria store e com as mesmas chaves, por isso as entradas existentes continuam legíveis e remover o pacote não exige limpar o cache. Tudo o resto — `add()`, contadores, `touch()`, locks e `Cache::tags()` — vai diretamente para a store, por isso o Laravel comporta-se exatamente como antes.

Algumas stores ficam nativas, porque o encaminhamento não as conseguiria reproduzir com exatidão; consulte a [configuração](./configuration.md#encaminhar-as-stores-existentes).

## O que corre sobre o driver `cacheer`

- `Cache::get()`, `put()`, `add()`, `remember()`, `flexible()`, `increment()`, `touch()`, `flush()` e o helper `cache()`.
- Locks atómicos: `Cache::lock()`, `restoreLock()`, `forceRelease()` e `refresh()`.
- Jobs únicos (`ShouldBeUnique`), o `withoutOverlapping()` do scheduler e comandos isoláveis (`--isolated`).
- Rate limiting (`RateLimiter`).
- O relógio de testes do Laravel: `$this->travel()` faz as entradas expirar.

A exceção é `Cache::tags()`; consulte [diferenças e limitações](./limitations.md).

## Páginas

- [Instalação](./installation.md): instalar o pacote, ver como correm as suas stores e adicionar uma store `cacheer`.
- [Configuração](./configuration.md): encaminhamento, backends, failover, cache em camadas, criptografia e prefixos.
- [Funcionalidades do Laravel](./framework-features.md): locks, filas, scheduler, rate limits e testes.
- [Diferenças e limitações](./limitations.md): o que se comporta de forma diferente das stores nativas do Laravel, nas stores encaminhadas e no driver `cacheer`.
- [Contribuir](../contribuicao/laravel.md): preparação, verificações do pacote e testes de integração Redis.
