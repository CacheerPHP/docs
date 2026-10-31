# CacheerPHP para Laravel

O CacheerPHP para Laravel (`cacheerphp/laravel`) é um driver de cache do Laravel suportado pelo CacheerPHP 6. A aplicação continua a usar `Cache::`, o helper `cache()`, os rate limiters, os jobs únicos e o scheduler. Por baixo, a store é o CacheerPHP, com failover, cache em camadas, criptografia em repouso e o [Cacheer Monitor](../cacheer-monitor/index.md).

Requisitos: PHP 8.3+, Laravel 12 ou 13 e CacheerPHP 6.

## Escolher como usar o CacheerPHP no Laravel

| Pretende… | Use |
| --- | --- |
| Que as funcionalidades de cache do Laravel (`Cache::`, locks, filas, scheduler) corram sobre o CacheerPHP | **O driver `cacheer`** — esta secção |
| Um cache CacheerPHP separado para os seus serviços, mantendo o cache do Laravel como está | [Um binding no container](../guias/laravel.md) |

Pode combinar os dois: o driver também regista o contrato `Cache` do CacheerPHP, para que os serviços o possam injetar e usar funcionalidades que `Cache::` não expressa, como a invalidação por tags.

## O que corre sobre o driver

- `Cache::get()`, `put()`, `add()`, `remember()`, `flexible()`, `increment()`, `touch()`, `flush()` e o helper `cache()`.
- Locks atómicos: `Cache::lock()`, `restoreLock()`, `forceRelease()` e `refresh()`.
- Jobs únicos (`ShouldBeUnique`), o `withoutOverlapping()` do scheduler e comandos isoláveis (`--isolated`).
- Rate limiting (`RateLimiter`).
- O relógio de testes do Laravel: `$this->travel()` faz as entradas expirar.

A exceção é `Cache::tags()`; consulte [diferenças e limitações](./limitations.md).

## Páginas

- [Instalação](./installation.md): instalar o pacote, adicionar uma store e confirmar que funciona.
- [Configuração](./configuration.md): backends, failover, cache em camadas, criptografia e prefixos.
- [Funcionalidades do Laravel](./framework-features.md): locks, filas, scheduler, rate limits e testes.
- [Diferenças e limitações](./limitations.md): o que se comporta de forma diferente das stores nativas do Laravel e a atualização a partir do pacote 0.1.
- [Contribuir](../contribuicao/laravel.md): preparação, verificações do pacote e testes de integração Redis.
