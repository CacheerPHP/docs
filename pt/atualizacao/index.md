# Guia de Atualização

Aqui você encontra o passo a passo para atualizar o CacheerPHP sem quebrar seu projeto.

## 1. Consulte o changelog

Antes de atualizar:

- Leia as notas de versão no GitHub
- Confira o `CHANGELOG.md`
- Veja PRs destacados com breaking changes

## 2. Atualize a biblioteca

```sh
composer update silviooosilva/cacheer-php
```

Se usa versão fixa no `composer.json`, ajuste o constraint primeiro (ex.: `^2.4` → `^2.5`).

Problemas com o lock? Limpe o cache e regenere:

```sh
composer clear-cache
rm composer.lock
composer update
```

## 3. Execute os testes

- `composer test`
- `composer lint`
- `composer analyse`

<!-- Se sua stack usa o Monitor, rode os cenários sintéticos:

```sh
php cacheer-monitor/Tests/stress_test.php
php cacheer-monitor/Tests/stress_io.php
```
-->

## 4. Atualize configs e assets

- Reaplique scripts de configuração se novas variáveis surgirem
- Limpe diretórios de cache (`rm -rf storage/cache` ou equivalente)
- Recompile bundles/frontends que dependam do CacheerPHP

<!--
## 5. Atualize o Cacheer Monitor

Dentro de `cacheer-monitor/`:

```sh
composer install
php bin/cacheer-monitor serve --host=127.0.0.1 --port=9966
```

A CLI do Monitor segue a mesma versão semântica da biblioteca; mantenha ambas alinhadas.
-->

## 6. Valide integrações

- Redis: confirme que os novos comandos funcionam
- Frameworks: execute testes de features (Laravel, Symfony etc.)
- Reporters customizados: cheque se logs/filas continuam OK

## 7. Plano de rollback

Guarde o lock e `vendor/` antigos até validar o upgrade. Em caso de falha:

```sh
git checkout composer.json composer.lock
composer install
```

## 8. Relate problemas

Abra uma issue com:

- Versão antes/depois
- Versão de PHP/SO
- Stack trace ou teste quebrado
- Passos para reproduzir

Manter o CacheerPHP atualizado garante correções e melhorias contínuas — obrigado por cuidar da sua stack!
