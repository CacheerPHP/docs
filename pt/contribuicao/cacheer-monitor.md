# Contribuir para o Cacheer Monitor

Abra uma issue ou um pull request em [CacheerPHP/monitor](https://github.com/CacheerPHP/monitor).
O Monitor é um pacote PHP com um dashboard estático; não requer uma compilação do frontend.

## Preparação e verificações

Precisa de PHP 8.3+, Composer e Node.js com npm para desenvolver o dashboard.

```sh
git clone https://github.com/CacheerPHP/monitor.git
cd monitor
composer install
npm ci
composer test
npm run test:ui
npm run lint
```

`composer test` executa o PHPUnit. Os testes PHP de comportamento ficam em
`Tests/Unit`; os diagnósticos e as regressões do dashboard em Node ficam em
`Tests/Regression`. `npm run test:ui` executa essas regressões, não testes visuais
num navegador. `npm run lint` verifica o JavaScript com ESLint. Não existe um
script `npm test`.

Use um ficheiro de eventos descartável e uma porta de loopback livre nas
verificações manuais. Aponte o reporter da aplicação e o servidor para o mesmo
ficheiro; consulte o [início rápido](../cacheer-monitor/quick-start.md). Não
inclua credenciais, dados capturados da aplicação nem logs gerados nos commits.
Verifique alterações ao dashboard nos dois temas, em ecrãs estreitos e com
navegação por teclado.
