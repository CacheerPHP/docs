# Bloqueios Distribuídos — `Cacheer::lock()`

*Novo na v5.2.0*

Um lock é uma guarda nomeada e mutuamente exclusiva suportada pelo driver de cache ativo. Use-o para garantir que só um processo executa uma secção crítica de cada vez — reconstruir um relatório, enviar um email único, drenar uma fila, etc.

```php
$lock = $cache->lock('rebuild-report', ttl: 30);

if ($lock->acquire()) {
    try {
        rebuildReport();
    } finally {
        $lock->release();
    }
}
```

`lock()` funciona numa instância **e** através da fachada estática:

```php
use Silviooosilva\CacheerPhp\Cacheer;

Cacheer::lock('rebuild-report', 30)->get(fn () => rebuildReport());
```

---

## Porquê e quando usar?

Em produção a sua aplicação **não é um programa a correr uma vez** — são muitos
processos PHP ao mesmo tempo (workers php-fpm a servir pedidos, workers de fila,
cron jobs), muitas vezes em **vários servidores**. Nenhum sabe que os outros
existem. Um lock é a forma de esses processos independentes se coordenarem
através do backend de cache partilhado para que:

> **só um processo execute um bloco de código de cada vez — em todo o lado.**

Use um lock sempre que *"se dois destes correrem ao mesmo tempo, algo corre mal"* —
trabalho duplicado, um efeito secundário duplicado ou um resultado corrompido —
especialmente numa operação **multi-passo** (verificar *e depois* agir) ou em algo
que nem sequer é da base de dados (enviar um email, chamar uma API, gerar um ficheiro).

### Exemplos do mundo real

| Cenário | Sem lock | Com lock |
|---------|----------|----------|
| **Cron em vários servidores** (email de resumo diário) | 3 servidores enviam-no → utilizadores recebem 3× | `lock('daily-digest')` — um servidor ganha, os outros saltam |
| **Reconstrução cara de cache** (dashboard, índice de pesquisa) | 200 misses simultâneos recalculam todos → a BD derrete | um pedido reconstrói, os restantes esperam pelo resultado |
| **Processamento de pagamentos / encomendas** | duplo clique ou retry → cobrança em dobro | `lock("invoice:$id")` — cobra uma vez |
| **Worker de background singleton** | cinco workers drenam a mesma fila | quem detém `lock('queue-drainer')` é "o único" |
| **API externa frágil** ("um pedido de cada vez") | chamadas concorrentes colidem | as chamadas alinham-se, uma de cada vez |
| **Setup único no arranque** | dez containers correm a migração | corre uma vez |

> O caso da "reconstrução cara de cache" é tão comum que o CacheerPHP trata dele
> por si: [`remember()`](./funcoes-cache.md) e `flexible()` já usam um lock
> internamente para prevenir stampedes. Normalmente só recorre à API `lock()`
> direta para **efeitos secundários que têm de acontecer uma única vez**, como
> nas linhas acima.

### Quando *não* usar

- **Contadores puros** (`+1` a um contador de visualizações) → use `increment()`, que já é atómico. Um lock é exagero.
- **Uma linha que tem de ser única** → uma constraint `UNIQUE` na base de dados é mais simples e mais rápida.
- **Cache de leitura simples** → basta fazer cache; a única parte que precisa de proteção é o recálculo, que `remember()` / `flexible()` já tratam.

---

## `lock()`

```php
$cache->lock(string $name, int $ttl = 60): \Silviooosilva\CacheerPhp\Support\CacheLock
```

Devolve um `CacheLock` ligado ao store ativo. `$ttl` é o tempo de vida máximo do lock em segundos — uma rede de segurança para que um detentor que crashe não bloqueie o lock para sempre.

Cada chamada devolve um `CacheLock` **novo** com o seu próprio *owner token* aleatório. Um lock só pode ser libertado pelo detentor que o adquiriu, pelo que um lock expirado e readquirido nunca é libertado por baixo do novo dono.

> Se o driver ativo não suportar locking, `lock()` lança `BadMethodCallException`. Os quatro drivers incluídos (File, Database, Redis, Array) suportam-no.

---

## Métodos do `CacheLock`

### `acquire()`

```php
$lock->acquire(): bool
```

Tenta adquirir o lock uma vez, sem esperar. Devolve `true` se adquirido, `false` se já está em uso.

### `release()`

```php
$lock->release(): bool
```

Liberta o lock — mas só se este detentor ainda for o dono. Devolve `false` se o lock nunca foi detido por esta instância (ou foi tomado após expirar).

### `block()`

```php
$lock->block(int $seconds, ?Closure $callback = null): mixed
```

Espera até `$seconds` pelo lock (por polling).

- **Sem** callback: devolve `true` se o lock foi adquirido dentro da janela, `false` caso contrário.
- **Com** callback: executa-o sob o lock e liberta-o no fim, devolvendo o resultado do callback — ou `false` se o lock nunca foi adquirido.

```php
// Espera até 5s e depois executa em exclusivo
$result = $cache->lock('export', 30)->block(5, fn () => generateExport());
```

### `get()`

```php
$lock->get(?Closure $callback = null): mixed
```

Como `block()`, mas **não-bloqueante** — uma única tentativa de aquisição.

- **Sem** callback: igual a `acquire()`.
- **Com** callback: executa-o sob o lock e liberta-o no fim, devolvendo o resultado — ou `false` se o lock não pôde ser adquirido.

```php
$result = $cache->lock('rebuild', 30)->get(fn () => rebuild());
if ($result === false) {
    // outro processo já está a reconstruir
}
```

### `owner()`

```php
$lock->owner(): string
```

Devolve o token aleatório que identifica este detentor. Útil para logging/depuração.

---

## Como cada driver implementa o locking

| Driver | Mecanismo | Alcance |
|--------|-----------|---------|
| **Redis** | `SET key value NX EX ttl`, libertado com um script Lua de compare-and-delete | Entre processos / entre servidores |
| **Database** | Uma tabela `cacheer_locks` cuja `PRIMARY KEY` no nome do lock é o portão atómico | Entre processos / entre servidores |
| **File** | `flock(LOCK_EX \| LOCK_NB)` num ficheiro por lock; liberta automaticamente se o processo morrer | Entre processos (mesmo servidor) |
| **Array** | Mapa em memória | Apenas no processo (testes / âmbito do pedido) |

Os drivers Redis, Database e File oferecem exclusão mútua **real entre processos**. O driver Array é local ao processo por natureza e destina-se a testes ou memoização dentro de um pedido.

---

## Padrões

### Single-flight (evitar trabalho duplicado)

```php
// Só um worker reconstrói o cache; os outros seguem em frente.
$cache->lock('cache:rebuild', 60)->get(function () use ($cache) {
    $cache->putCache('home', renderHomepage(), ttl: 600);
});
```

### Esperar-e-executar

```php
// Fica na fila atrás de quem detém o lock, até 10 segundos.
$cache->lock('invoice:'.$id, 30)->block(10, fn () => chargeInvoice($id));
```

### Acquire/release manual

```php
$lock = $cache->lock('migration', 120);

if (! $lock->acquire()) {
    exit("Já há outra migração em curso.\n");
}

try {
    runMigration();
} finally {
    $lock->release();
}
```

---

## Notas

- Os TTLs de lock são best-effort nos drivers Database e File (dependem do relógio local); escolha um `$ttl` confortavelmente maior do que a secção crítica.
- O driver File mantém um pequeno ficheiro por nome de lock em `cacheer-locks/` dentro do diretório de cache. Estes ficheiros são reutilizados, não apagados no release — é intencional e necessário para a exclusão mútua correta.
- Os locks partilham o backend de cache mas vivem num keyspace separado, pelo que nunca colidem com os seus valores em cache.
- Os métodos atómicos `increment()` / `decrement()` assentam nesta mesma camada de locking.
