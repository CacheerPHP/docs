# Guia de Contribuição

Obrigado por ajudar a melhorar o CacheerPHP. Esta página cobre o fluxo de
desenvolvimento da v6. Para a política completa, veja `CONTRIBUTING.md`,
`SECURITY.md` e o `ROADMAP.md` no repositório do pacote.

## Setup

```sh
git clone https://github.com/silviooosilva/CacheerPHP
cd CacheerPHP
composer install
```

Requer PHP 8.3+. O núcleo não tem extensões obrigatórias; instale `pdo_sqlite`,
`openssl` e `zlib` para rodar a suíte local completa, e `predis/predis` para Redis.

## Suítes de teste

Os testes usam [Pest](https://pestphp.com/) e são separados por área:

```sh
composer test            # suíte unitária sem serviços (padrão)
composer test:kernel     # núcleo v6 (Cache, adaptadores, CLI, rehearsals)
composer test:contract   # conformidade das stores (Array, File)
composer test:storage    # pipeline de armazenamento / envelope
composer test:concurrency  # harnesses de contenção de locks e contadores
composer test:integration  # Redis / banco (requer serviços)
composer test:all        # tudo
```

Data providers devem usar a forma de atributo, não a anotação:

```php
#[\PHPUnit\Framework\Attributes\DataProvider('cases')]
```

## Análise estática e estilo

```sh
composer analyse   # PHPStan nível 5, sem supressões
composer lint      # php-cs-fixer (dry-run)
composer fix       # php-cs-fixer (aplica)
```

## Tempo determinístico

Nunca chame `time()` ou `sleep()` em testes ou no código de uma store. O tempo
passa por um `Clock` injetado; os testes avançam um `FakeClock`. Isso mantém a
suíte rápida e o comportamento de expiração/stale exato.

## Adicionando uma store

Implemente o contrato `Store`, adicione apenas as capacidades que você garante e
comprove estendendo `Tests\Support\StoreConformance`. Veja o
[guia de stores personalizadas](../guias/stores-personalizadas.md). Uma store nunca
deve varrer ou limpar dados fora do seu keyspace configurado.

## Portões de qualidade de um pull request

- Mapeia para um item do roadmap (e um RFC aceito, para mudanças substanciais).
- Comportamento público tem testes unitários ou de contrato; comportamento
  específico de driver tem testes de integração; alegações de concorrência têm
  testes de contenção.
- Nova configuração é tipada e documentada; novos eventos carregam apenas metadados
  (nunca valores).
- Sem efeitos colaterais ocultos de filesystem, ambiente, timezone, schema ou rede.
- `composer analyse` e `composer lint` passam.
- Exemplos e notas de migração relevantes são atualizados.
- Mudanças sensíveis a performance incluem evidência de benchmark antes/depois.

## Reportando

- **Bugs / features:** use os templates de issue do repositório.
- **Segurança:** não abra uma issue pública — siga o `SECURITY.md`.
- **Mudanças substanciais:** abra um RFC primeiro para acordar o design antes do
  código.
