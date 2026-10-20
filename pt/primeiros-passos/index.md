# Primeiros Passos

**Cache para PHP independente de framework, com invalidação por escopo, stores componíveis e monitorização opcional.** Este exemplo guarda um pequeno catálogo, mostra um hit entre dois processos PHP e atualiza o resultado quando a origem muda.

## 1. Instalar

Precisa de **PHP 8.3+** e Composer. A store de ficheiros não precisa de Redis, base de dados ou extensões PHP opcionais.

```sh
mkdir cacheer-demo
cd cacheer-demo
composer require silviooosilva/cacheer-php:"^6.0"
```

Já tem um projeto PHP? Execute apenas o comando Composer na sua raiz. A restrição `^6.0` mantém as atualizações na linha estável 6.x.

## 2. Adicionar uma origem e um cache

Crie `products.json` na raiz do projeto:

```json
[
  {"id": 1, "name": "Desk lamp"},
  {"id": 2, "name": "Notebook"}
]
```

Guarde este código em `cache-demo.php`, ao lado do JSON:

```php
<?php

declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Silviooosilva\CacheerPhp\Cacheer;

$catalog = Cacheer::file(__DIR__ . '/var/cache')->scope('catalog');

if (in_array('--refresh', $argv, true)) {
    $catalog->delete('products');
}

$products = $catalog->remember('products', 300, static function (): array {
    echo "Reading products.json...\n";

    return json_decode(
        file_get_contents(__DIR__ . '/products.json'),
        true,
        flags: JSON_THROW_ON_ERROR,
    );
});

echo 'Products: ' . count($products) . PHP_EOL;
```

O callback é a origem dos dados. Na aplicação, substitua a leitura do JSON por uma consulta à base de dados ou uma chamada à API. Construa o cache uma vez no bootstrap e injete-o nos serviços que precisam dele. Mantenha `var/cache` com permissão de escrita e fora da raiz pública.

## 3. Ver um miss e um hit

```sh
php cache-demo.php
php cache-demo.php
```

Primeira execução:

```text
Reading products.json...
Products: 2
```

Segunda execução, dentro de cinco minutos:

```text
Products: 2
```

O segundo processo lê o resultado guardado sem executar o callback. A store de ficheiros persiste entre pedidos e execuções CLI. `Cacheer::inMemory()` dura apenas durante a vida do seu processo PHP.

## 4. Alterar a origem e invalidar

Adicione um terceiro produto ao `products.json`. Execute `php cache-demo.php`: continua a indicar dois produtos enquanto o resultado em cache for válido.

```sh
php cache-demo.php --refresh
```

O comando remove apenas `products` no escopo `catalog`. O callback volta a ler o ficheiro e indica três produtos. Na aplicação, invalide a chave depois de a alteração na origem ter sucesso. `$catalog->clear()` limpa esse escopo; mantenha uma instância raiz se também precisar de limpar toda a store configurada.

## Continuar na aplicação

| A sua aplicação | Próximo passo |
| --- | --- |
| PHP puro ou um worker | Crie o cache no bootstrap e injete `Contracts\Cache` nos serviços. O exemplo acima está pronto a executar. |
| Laravel | [Registe o cache num service provider e injete-o num serviço de catálogo](../guias/laravel.md). |
| Ainda está a decidir | [Compare CacheerPHP com Laravel Cache](../guias/cacheerphp-vs-laravel-cache.md). |
| Quer ver hits e misses | [Ligue o Cacheer Monitor](../cacheer-monitor/quick-start.md). |

## Escolher armazenamento quando precisar

Comece por ficheiros num único servidor. Escolha [Redis](../api/drivers.md) ou uma [store de base de dados](../api/drivers.md) quando processos em vários servidores precisarem de partilhar o backend. Forneça a ligação; crie explicitamente o schema da store de base de dados durante a configuração. Para testes e tarefas CLI curtas, use `Cacheer::inMemory()`.

## Algumas regras a manter

- `get()` devolve o valor original. `null`, `false`, `0` e arrays vazios em cache são hits válidos; use `has()` ou `entry()->isHit()` quando a presença importar.
- O TTL aceita segundos positivos, uma string como `'5 minutes'`, `DateInterval` ou `Ttl`. TTL omitido/null usa a política predefinida, se existir; caso contrário, não expira. Use `forever()` para declarar armazenamento sem expiração.
- `remember()` coordena misses quando a store suporta locks. A expiração da lease e timeouts podem permitir trabalho duplicado; os callbacks devem tolerá-lo.
- `flexible()` suporta stale-while-revalidate. O executor predefinido executa inline; [configure execução diferida](../guias/stale-while-revalidate.md) antes de esperar atualizações depois da resposta.

Consulte as [assinaturas da API](../api/index.md), o [suporte de versões](../atualizacao/suporte-de-versoes.md) ou o [guia de migração](../atualizacao/index.md) se estiver a atualizar da v5.
