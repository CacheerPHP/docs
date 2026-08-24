# Escopos

Um **escopo** é um keyspace isolado dentro de uma store. Escopos substituem os
namespaces posicionais por string da v5 por um [`Scope`](../api/construtor-de-opcoes.md#scope)
tipado e aninhável que você pode limpar separadamente.

## Básico

```php
$reports = $cache->scope('reports');
$reports = $cache->in('reports');    // in() é um alias — use o que ler melhor

$reports->set('daily', $rows);
$reports->get('daily');          // as linhas
$cache->get('daily');            // null — o escopo raiz é outro keyspace
```

O mesmo nome de chave vive de forma independente em cada escopo:

```php
$cache->scope('reports')->set('daily', $reportRows);
$cache->scope('billing')->set('daily', $invoice);

$cache->scope('reports')->get('daily'); // $reportRows
$cache->scope('billing')->get('daily'); // $invoice
```

## Limpando um escopo

`clear()` em um cache com escopo remove apenas aquele escopo — nunca a store inteira:

```php
$cache->scope('reports')->clear();   // apenas entradas de "reports"
$cache->clear();                     // a store inteira (raiz)
```

Isso requer que a store implemente `FlushableScopeStore` (as quatro nativas
implementam). Em uma store sem isso, um `clear()` com escopo lança
`UnsupportedCapabilityException` em vez de limpar demais.

## Aninhamento

Escopos aninham em qualquer profundidade (até 64 segmentos):

```php
$tenant = $cache->scope('tenant:42');
$tenant->scope('reports')->set('daily', $rows);

// Limpar o pai limpa os filhos também.
$tenant->clear();
```

## Validação

Segmentos de escopo são validados: não podem ser vazios, exceder 255 bytes ou conter
barras ou caracteres de controle (`InvalidScopeException`).

## Escopos vs. tags

- **Escopos** são estruturais: uma entrada pertence a exatamente um escopo, e você
  limpa um escopo inteiro de uma vez. Use-os para separar funcionalidades, tenants ou
  ambientes.
- **[Tags](../tutoriais/exemplo-10-tagging.md)** são rótulos transversais: uma entrada
  pode ter muitas tags, e você invalida por tag. Use-as para relações que cruzam
  escopos.

## O escopo vale para tudo, não só get/set

Contadores, tags e locks também têm escopo, então dois escopos não colidem:

```php
$cache->in('tenant-a')->increment('signups', 1, initial: 0);
$cache->in('tenant-b')->increment('signups', 5, initial: 0);
// → 1 e 5, em keyspaces separados

$cache->in('tenant-a')->tag('p1', 'products');
$cache->in('tenant-b')->tag('p1', 'products');
$cache->in('tenant-a')->flushTag('products');   // não afeta o tenant-b

$a = $cache->in('tenant-a')->lock('nightly-import', 300);
$b = $cache->in('tenant-b')->lock('nightly-import', 300);
// ambos adquirem — mesmo nome, escopos diferentes
```

## Tudo é um escopo internamente

O cache raiz é apenas o escopo raiz, e criar um escopo retorna o **mesmo tipo**:

```php
$cache->scope('reports') instanceof Cacheer;   // true
```

Ou seja, um cache com escopo nunca perde parte da API silenciosamente — ele tem
`remember()`, `flexible()`, leituras em lote, capacidades, `withPolicy()` e
`formatted()` como o cache raiz. Pergunte em qual escopo você está com
`boundScope()`:

```php
$cache->in('tenant:42')->in('reports')->boundScope();   // "tenant:42/reports"
```
