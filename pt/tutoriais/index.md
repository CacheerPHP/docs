# Tutoriais

Exemplos curtos e objetivos para o CacheerPHP 6. Cada um é autocontido e usa apenas o
pacote núcleo. Para aprofundamentos, veja os [Guias](../guias/configuracao.md); para
assinaturas precisas, a [Referência da API](../api/index.md).

## Básico

1. [Cache simples de dados](./exemplo-01-cache-simples.md) — `set` / `get` / `has` / `delete`
2. [Expiração personalizada (TTL)](./exemplo-02-tempo-expiracao-personalizado.md) — todos os formatos de TTL
3. [Limpeza e flush](./exemplo-03-limpeza-e-flush.md) — `delete`, `clear`, limpeza por escopo
4. [Escopos](./exemplo-04-namespaces.md) — keyspaces isolados
5. [Cacheando uma resposta de API](./exemplo-05-cache-resposta-api.md) — `remember()`
6. [O objeto CacheEntry](./exemplo-06-saida-json.md) — hit/miss, timestamps, TTL

## Trabalhando com dados

7. [Operações em lote](./exemplo-07-saida-array.md) — `many` / `setMany` / `deleteMany`
8. [Criptografia e compressão](./exemplo-08-saida-string.md) — o pipeline de armazenamento
9. [Removendo entradas expiradas](./exemplo-09-limpeza-automatica.md) — `prune()` e a CLI
10. [Tagging](./exemplo-10-tagging.md) — agrupar e invalidar por tag
11. [Adaptador PSR-16](./exemplo-11-adaptador-psr16.md) — SimpleCache padrão
12. [TTL com DateInterval](./exemplo-12-dateinterval-ttl.md) — intervalos nativos do PHP
13. [Valores falsy e null](./exemplo-13-valores-falsy.md) — um `null` cacheado é um hit

## Avançado

14. [Escritas condicionais e contadores atômicos](./exemplo-14-armazenamento-condicional-add.md) — `add()`, contadores, locks
15. [Observabilidade: eventos e métricas](./exemplo-15-estatisticas-instancia.md)
16. [Cache em camadas (L1/L2)](./exemplo-16-integracao-monitor.md)
17. [Locks e seções críticas](./exemplo-17-locks-contadores-atomicos.md)
18. [Proteção contra estampede e stale-while-revalidate](./exemplo-18-protecao-stampede-swr.md)

Veja também os guias [Store resiliente](../guias/store-resiliente.md) e
[Políticas](../guias/politicas.md).
