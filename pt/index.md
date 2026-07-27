# Documentação CacheerPHP (PT)

**Versão atual: 6.0** | PHP 8.3+ | [Guia de atualização](./atualizacao/index.md)

O CacheerPHP 6 é um cache **baseado em instâncias**: um núcleo `Cache` enxuto
sobre um contrato `Store` mínimo de quatro métodos, com tudo o mais — lotes,
tags, locks, contadores atômicos, camadas, resiliência, criptografia — como uma
**capacidade opcional** que você habilita. Não há estado global nem efeito
colateral no autoload. Código v5 continua funcionando pela ponte `LegacyCacheer`.

## Novidades da v6.0

- **Núcleo baseado em instâncias.** `Cache` explícito e `ScopedCache` imutável
  sobre `Key`, `Scope`, `Ttl` e `CacheEntry` tipados; o tempo é um `Clock` injetado.
- **Núcleo pequeno, capacidades honestas.** Uma store implementa quatro métodos;
  o resto é declarado por interface e verificado em tempo de execução.
- **Decorators componíveis.** Camadas (L1/L2), resiliente (fallback com circuit
  breaker) e instrumentado (eventos tipados + métricas).
- **Proteção contra estampede.** `remember()` de execução única e
  `flexible()` (stale-while-revalidate) embutidos.
- **Armazenamento autenticado.** serialize → gzip opcional → AES-256-GCM opcional
  em um envelope versionado e à prova de adulteração.
- **Padrões e ferramentas.** Adaptadores PSR-16 e PSR-6, log PSR-3, ponte PSR-14
  e uma CLI de operações `cacheer`.
- **Migração.** Ponte `LegacyCacheer`, conjunto Rector opcional e reescrita na
  leitura de payloads v5. Veja o [guia de atualização](./atualizacao/index.md).

## Seções

- [Primeiros Passos](./primeiros-passos/index.md)
- [Guias](./guias/configuracao.md)
- [API](./api/index.md)
- [Tutoriais](./tutoriais/index.md)
- [Guia de Contribuição](./contribuicao/index.md)
- [Guia de Atualização](./atualizacao/index.md)
<!-- - [Monitor](./monitor/index.md) -->

Contribuições são bem-vindas. Veja o README na raiz para estrutura e orientações.
