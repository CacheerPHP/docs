# Referência da API — Construtor de Tempo (TimeBuilder)

O TimeBuilder oferece uma forma fluente (encadeável) de definir períodos de tempo de forma intuitiva e sem erros de digitação.

Ele permite informar `expirationTime` e `flushAfter` diretamente como inteiros ou usando métodos encadeados como `day(1)`, `week(2)`, etc.

O TimeBuilder pode ser encadeado a partir dos builders de File, Redis e Database para definir valores de `expirationTime()` e `flushAfter()`.

#### Uso simples (File)

```php
OptionBuilder::forFile()
    ->expirationTime('1 day')
    ->build();
```
Ou usando a abordagem encadeada do TimeBuilder:

```php
OptionBuilder::forFile()
    ->expirationTime()->day(1)
    ->build();
```

#### Uso com Redis e Database

```php
// TTL padrão de 10 minutos no Redis
OptionBuilder::forRedis()
  ->expirationTime()->minute(10)
  ->build();

// Intervalo de flush de 1 semana no Database
OptionBuilder::forDatabase()
  ->flushAfter()->week(1)
  ->build();
```

#### Métodos disponíveis

Cada método define um intervalo específico.

| Método          | Descrição                     | Exemplo        |
|-----------------|-------------------------------|----------------|
| `second($v)`    | Define o tempo em segundos    | `->second(30)` |
| `minute($v)`    | Define o tempo em minutos     | `->minute(15)` |
| `hour($v)`      | Define o tempo em horas       | `->hour(3)`    |
| `day($v)`       | Define o tempo em dias        | `->day(7)`     |
| `week($v)`      | Define o tempo em semanas     | `->week(2)`    |
| `month($v)`     | Define o tempo em meses       | `->month(1)`   |
| `year($v)`      | Define o tempo em anos        | `->year(1)`    |

#### Exemplo completo

```php
$Options = OptionBuilder::forFile()
    ->dir(__DIR__ . '/cache')
    ->expirationTime()->week(1)
    ->flushAfter()->minute(30)
    ->build();

var_dump($Options);
```

Saída esperada

```php
[
    "cacheDir" => "/path/to/cache",
    "expirationTime" => "1 week",
    "flushAfter" => "30 minutes"
]
```

Agora você pode definir tempos de expiração e limpeza sem precisar lembrar strings exatas. 🚀

