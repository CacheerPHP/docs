# Guia de Contribuição

Obrigado por ajudar a evoluir o CacheerPHP! Este guia explica como preparar o ambiente, seguir as convenções do projeto e enviar contribuições sem dor de cabeça.

## Pré-requisitos

- PHP 8.1 ou superior
- Composer 2+
- Redis (opcional) para testar recursos específicos
- Node.js/npm caso vá mexer nos assets do site de documentação

## Estrutura do repositório

```
CacheerPHP-Org/
├── CacheerPHP/            # Biblioteca principal
│   ├── src/
│   ├── tests/
│   └── ...
├── cacheerphp.github.io/  # Site publicado (gerado a partir de /docs)
└── docs/                  # Markdown fonte (inglês e português)
```

**Importante**: atualize a documentação na pasta `/docs`. O site (`cacheerphp.github.io/resources/docs`) é derivado desse conteúdo.

## Preparando o ambiente

```sh
# Clone o monorepo (contém biblioteca e docs)
git clone https://github.com/silviooosilva/CacheerPHP-Org.git
cd CacheerPHP-Org/CacheerPHP

# Instalar dependências PHP
composer install

# Executar testes
composer test

# Opcional: análise estática e lint
composer lint
composer analyse
```

<!--
## Rodando o monitor (opcional)

```sh
cd cacheer-monitor
composer install
php bin/cacheer-monitor serve --host=127.0.0.1 --port=9966
```

Use `CACHEER_MONITOR_EVENTS` para controlar o JSONL monitorado.
-->

## Branches e commits

- Crie uma branch específica (`feat/...`, `fix/...`, etc.).
- Commits curtos, com mensagens no imperativo (`feat: adiciona gerente redis`).
- Rebase com o `main`/`master` antes de abrir o PR.

## Checklist de testes

- `composer test`
- `composer lint`
- `composer analyse`
<!-- - Scripts relevantes do monitor (ex.: `php Tests/stress_io.php`) quando mexer em agregação/reporters -->

## Atualizando documentação

- Conteúdo em inglês → `docs/en/...`
- Traduções em português → `docs/pt/...`
- Se só existir uma língua, adicione links cruzados
- Inclua novos tópicos no `index.md` correspondente

## Enviando Pull Request

1. Faça push da branch para seu fork.
2. Abra PR contra `silviooosilva/CacheerPHP` (ou repositório apropriado).
3. Preencha o template: descrição, testes executados, evidências se houver UI.
4. Responda aos feedbacks; faça squash ou rebase quando solicitado.

## Convenções de código

- PSR-12 como base, salvo quando o arquivo já segue uma variante
- Use `declare(strict_types=1);` e tipagem sempre que possível
- Evite novas dependências sem discutir via issue
<!-- - No monitor (frontend), mantenha utilitários Tailwind e evite estilos inline -->

## Reportando issues

- Use o GitHub Issues
- Especifique ambiente, passos de reprodução, comportamento esperado vs observado
<!-- - Anexe logs, stack trace ou amostras JSONL se o problema envolver o monitor -->

Obrigado por fortalecer o CacheerPHP!
