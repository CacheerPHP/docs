# Versões e suporte

Escolha a documentação que corresponde à versão instalada na aplicação.

## Linhas atuais

| Linha | Estado | PHP | Instalação |
|---|---|---|---|
| 6.x | Prévia de desenvolvimento; a API baseada em instâncias documentada aqui | 8.3+ | `composer require silviooosilva/cacheer-php:"6.x-dev"` |
| 5.x | Linha estável publicada; a versão mais recente é a 5.2.0 | 8.2+ | `composer require silviooosilva/cacheer-php:"^5.2"` |

A última versão estável publicada é a [v5.2.0, de 6 de julho de 2026](https://github.com/CacheerPHP/CacheerPHP/releases/tag/v5.2.0).
O estado foi verificado em 3 de outubro de 2026. Consulte as [versões no GitHub](https://github.com/CacheerPHP/CacheerPHP/releases) para publicações e registos de alterações mais recentes.

O ramo de desenvolvimento 6.x pode mudar antes de uma versão estável. É necessário indicar a restrição Composer para experimentar a API desta documentação; uma instalação sem versão seleciona atualmente o pacote estável 5.x.

## Suporte durante a migração

O [guia de migração da v6](./index.md#janela-de-suporte) define o período de suporte: a v6 é a linha em desenvolvimento ativo e a v5 recebe correções de segurança e de funcionamento durante 12 meses após a publicação da versão estável 6.0. Esse período começa quando a 6.0 estável for publicada.

Use a [documentação da v5](../../../v5/pt/primeiros-passos/index.md) numa aplicação estável existente. Consulte o [guia de migração](./index.md) para preparar a atualização para a API baseada em instâncias.

## Reportar um problema

Abra um [problema no GitHub](https://github.com/CacheerPHP/CacheerPHP/issues) com as versões do CacheerPHP e do PHP, a store escolhida, um exemplo mínimo e o comportamento esperado. Para problemas de integração com o Monitor, inclua o resultado de `vendor/bin/cacheer-monitor doctor`.

O projeto é mantido por [Silvio Silva](https://github.com/silviooosilva). O [guia de contribuição](../contribuicao/index.md) explica como colaborar.
