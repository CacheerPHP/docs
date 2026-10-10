# Versões e suporte

Escolha a documentação que corresponde à versão instalada na aplicação.

## Linhas atuais

| Linha | Estado | PHP | Instalação |
|---|---|---|---|
| 6.x | Linha estável atual; começa na 6.0.0 | 8.3+ | `composer require silviooosilva/cacheer-php:"^6.0"` |
| 5.x | Manutenção; a versão mais recente é a 5.2.0 | 8.2+ | `composer require silviooosilva/cacheer-php:"^5.2"` |

O CacheerPHP 6.0.0 é a versão estável da API baseada em instâncias documentada aqui.
A restrição `^6.0` aceita atualizações estáveis da linha 6.x sem mudar de versão principal. Consulte as [versões no GitHub](https://github.com/CacheerPHP/CacheerPHP/releases) para notas de lançamento e registos de alterações.

## Suporte durante a migração

O [guia de migração da v6](./index.md#janela-de-suporte) define o período de suporte: a v6 recebe funcionalidades e correções de funcionamento e segurança. A v5 recebe apenas correções de segurança e de funcionamento durante 12 meses a partir do lançamento da 6.0.0; não recebe novas funcionalidades.

Use a [documentação da v5](../../../v5/pt/primeiros-passos/index.md) numa aplicação que ainda execute a v5. Siga o [guia de migração](./index.md) para atualizar para a linha estável 6.x.

## Reportar um problema

Abra um [problema no GitHub](https://github.com/CacheerPHP/CacheerPHP/issues) com as versões do CacheerPHP e do PHP, a store escolhida, um exemplo mínimo e o comportamento esperado. Para problemas de integração com o Monitor, inclua o resultado de `vendor/bin/cacheer-monitor doctor`.

O projeto é mantido por [Silvio Silva](https://github.com/silviooosilva). O [guia de contribuição](../contribuicao/index.md) explica como colaborar.
