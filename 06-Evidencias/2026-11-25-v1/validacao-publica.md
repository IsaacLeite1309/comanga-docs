# Validação manual parcial do site público — 25/11/2026

Ambiente: navegador em sessão anônima; site `https://www.comanga.com.br`. A versão exibida no documento é 1.0; esta observação não identifica por si só os commits em produção.

| Ação | Resultado observado |
| --- | --- |
| Abrir a pesquisa de obras | Catálogo carregado com 932 obras e paginação de 39 páginas. O primeiro carregamento apresentou estado de espera antes de exibir os resultados. |
| Pesquisar `One Piece` | A busca exibiu 8 obras, incluindo `One Piece` como Mangá, `One Piece Blue Deep` como Databook e `One Piece Novel A` como Light Novel. |
| Aplicar o filtro `Databook` à busca `One Piece` | Foram exibidas 5 obras, todas identificadas como Databook; o termo permaneceu na pesquisa. |
| Abrir `One Piece` | A ficha apresentou título, autor Eiichiro Oda, sinopse, classificações e três edições brasileiras, com prévias de volumes. |
| Abrir a 3ª edição | Foram exibidos editora Panini, formato, acabamento, miolo e 37 volumes. |
| Abrir o volume 1 | A ficha apresentou capa, data de lançamento 02/04/2022, preço R$ 49,90, 614 páginas, ISBN-10 e ISBN-13. |
| Tentar adicionar o volume à coleção sem sessão | O sistema exibiu “Faça login para poder adicionar este volume à sua coleção.” e a caixa permaneceu desmarcada. |
| Enviar formulário de cadastro vazio | A tela pediu nome de usuário, e-mail, data válida, senha, confirmação e aceite dos termos; não foi criada conta. |
| Abrir coleção e lista de desejos sem sessão | Ambas as páginas solicitaram login para gerenciar os respectivos itens. |
| Abrir Lançamentos | A tela carregou a lista do mês, com 22 volumes no momento da consulta, filtros e acesso às fichas. |
| Pesquisar `Chainsaw` em Lançamentos | Foi exibido 1 volume: `Chainsaw Man`, volume 24. Ao selecionar o mês anterior, a URL passou a indicar outubro de 2026 e preservou o termo. |
| Abrir a página de Eiichiro Oda | A página identificou o autor e listou 9 obras publicadas no Brasil, incluindo `One Piece`. |

Esta verificação é parcial; não criou conta nem alterou acervo. Não inclui login, cadastro válido, envio de e-mail, acervo autenticado, administração ou execução integral dos casos CT-001 a CT-028. Não foram registradas capturas de tela.
