# Exemplo 14 — Escrita Condicional com add()

*Corrigido na v5.0.0*

O método `add()` guarda um valor **apenas se a chave ainda não existir**. Devolve `true` quando o valor foi guardado e `false` quando a chave já existia.

> **Alteração incompatível:** Na v4.x os valores de retorno estavam invertidos. Ver o [guia de migração](../atualizacao/v5-migration.md) para detalhes.

## Uso Básico

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$cache = new Cacheer();
$cache->setDriver()->useArrayDriver();

// Primeira chamada — a chave não existe, o valor é guardado
$stored = $cache->add('lock', getmypid(), ttl: 30);
var_dump($stored); // true

// Segunda chamada — a chave já existe, nada acontece
$stored = $cache->add('lock', 99999, ttl: 30);
var_dump($stored); // false

// O valor original é preservado
echo $cache->getCache('lock'); // imprime o primeiro PID
```

## Caso de Uso: Lock Simples

```php
function acquireLock(Cacheer $cache, string $resource, int $ttlSeconds = 10): bool
{
    return $cache->add("lock:{$resource}", getmypid(), ttl: $ttlSeconds);
}

function releaseLock(Cacheer $cache, string $resource): void
{
    $cache->clearCache("lock:{$resource}");
}

if (acquireLock($cache, 'report-generation')) {
    try {
        // Só um processo executa isto de cada vez
        generateReport();
    } finally {
        releaseLock($cache, 'report-generation');
    }
} else {
    echo "Outro processo já está a gerar o relatório.";
}
```

## Caso de Uso: Definições por Omissão

```php
// Inicializa padrões sem substituir personalizações do utilizador
$cache->add('settings:theme', 'light');
$cache->add('settings:language', 'pt');
$cache->add('settings:per_page', 25);

// Se o utilizador já escolheu "dark", a linha acima não faz nada
echo $cache->getCache('settings:theme'); // "dark" (escolha do utilizador preservada)
```
