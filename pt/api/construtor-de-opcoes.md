# Objetos de valor — Key, Scope, Ttl, CacheEntry

> O `OptionBuilder` da v5 acabou. A v6 configura stores por
> [construtores nomeados](./funcoes-cache.md#construtores-nomeados) e um
> [`PipelineConfig`](./configuracao.md) tipado. Os objetos de valor abaixo
> substituem os argumentos por string que a v5 passava.

Os quatro objetos ficam em `Silviooosilva\CacheerPhp\Kernel`. São imutáveis e
validados na construção.

## `Key`

Uma chave de cache validada, opcionalmente ligada a um [`Scope`](#scope).

```php
use Silviooosilva\CacheerPhp\Kernel\Key;

Key::named(string $value): self   // lança InvalidKeyException se vazia, >1024 bytes ou com caracteres de controle

$key->value(): string
$key->scope(): Scope
$key->within(Scope $scope): self
$key->identity(): string
```

Todo método de `Cacheer` aceita também uma string simples; ela é embrulhada como
`Key::named($string)` no escopo do chamador.

## `Scope`

Um keyspace isolado. Escopos são listas ordenadas de segmentos; o escopo raiz é
vazio.

```php
use Silviooosilva\CacheerPhp\Kernel\Scope;

Scope::root(): self
Scope::named(string $segment): self
Scope::fromSegments(iterable $segments): self

$scope->child(string $segment): self
$scope->append(Scope $other): self
$scope->isRoot(): bool
$scope->segments(): array
$scope->contains(Scope $other): bool
```

Segmentos não podem ser vazios, exceder 255 bytes ou conter barras ou caracteres de
controle (`InvalidScopeException`). Um escopo tem no máximo 64 segmentos. Veja o
[guia de Escopos](../guias/escopos.md).

## `Ttl`

Um time-to-live normalizado. Veja [TTL e Clock](./construtor-de-tempo.md) para as
formas de entrada e a semântica de "para sempre".

```php
use Silviooosilva\CacheerPhp\Kernel\Ttl;

Ttl::forever(): self
Ttl::seconds(int): self            // deve ser > 0
Ttl::minutes(int) / ::hours(int) / ::days(int) / ::weeks(int): self
Ttl::until(DateTimeInterface $when, Clock $clock): self
Ttl::from(Ttl|DateInterval|int|string|null): self

$ttl->isForever(): bool
$ttl->inSeconds(): ?int            // null quando para sempre
$ttl->expiresAt(Clock $clock): ?int  // ts unix absoluto, ou null quando para sempre
```

## `CacheEntry`

O resultado de uma leitura. Distingue um **miss** de um `null` cacheado.

```php
$entry = $cache->entry('user:42');

$entry->isHit(): bool
$entry->isMiss(): bool
$entry->value(): mixed                 // lança CacheMissException num miss
$entry->valueOr(mixed $default): mixed
$entry->createdAt(): ?int              // ts unix da escrita
$entry->expiresAt(): ?int              // ts unix, ou null quando para sempre
$entry->isExpired(Clock $clock): bool
$entry->remainingTtl(Clock $clock): ?int  // segundos restantes, null quando para sempre, 0 num miss
```

Um `null` cacheado retorna `isHit() === true` e `value() === null` — algo que
`get()` não consegue expressar, e por isso `entry()` existe.
