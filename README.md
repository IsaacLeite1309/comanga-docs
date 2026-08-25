# CoMangá Docs

Repositório da documentação técnica e acadêmica do CoMangá, plataforma para catalogação e gerenciamento de coleções físicas de mangás. Ele é a fonte de verdade para requisitos, regras de negócio, arquitetura, diagramas, critérios de aceite e planejamento de evolução.

## O sistema documentado

O CoMangá é composto por uma SPA React hospedada na Vercel e uma API REST Node.js/Express hospedada na Render. A API utiliza PostgreSQL no Neon, Prisma ORM, sessões stateful em cookie HttpOnly e Cloudflare R2 para capas internas processadas.

```text
React/Vercel -> API REST/Render -> PostgreSQL/Neon
                              -> Cloudflare R2
                              -> SMTP
```

Até o momento, estão implementados autenticação e sessões, administração de usuários/opções, Obras, Edições, Volumes, importação interna de capas, vitrine pública, detalhes públicos e listagem de Obras por Autor. Calendário, Estante Digital, Lista de Desejos, enriquecimento autenticado e a evolução para uma arquitetura distribuída permanecem planejados.

## Estrutura

| Diretório | Conteúdo |
| --- | --- |
| [`01-SERS`](./01-SERS/) | Especificação de Requisitos de Software: requisitos funcionais e não funcionais, regras de negócio e restrições. |
| [`02-User-Stories`](./02-User-Stories/) | Histórias de usuário e cenários Gherkin que orientam aceite e testes. |
| [`03-ATAM`](./03-ATAM/) | Cenários de análise arquitetural e atributos de qualidade. |
| [`04-DAS`](./04-DAS/) | Documento de Arquitetura de Software, com decisões efetivamente adotadas. |
| [`05-Diagramas`](./05-Diagramas/) | Casos de uso, sequência, DER físico e classes ORM Prisma. |
| [`06-Planejamento`](./06-Planejamento/) | Kanban, Definition of Done, sprints e plano geral de desenvolvimento. |

## Artefatos principais

- **SERS**: referência principal para comportamento esperado, segurança, privacidade e regras do domínio.
- **User Stories e Gherkin**: tradução dos requisitos em cenários verificáveis de implementação.
- **DAS**: apresenta a arquitetura cliente-servidor, o monólito modular atual, sessões, persistência, mídia, testes, deploy e limites conhecidos.
- **ATAM**: apoia decisões relativas a desempenho, disponibilidade, confiabilidade, segurança e manutenibilidade.
- **Diagramas**: mantêm a visão visual do domínio, dados e fluxos relevantes.
- **Planejamento**: organiza os cartões por sprints, com critérios de aceite e o DoD adotado no projeto.

## Como a documentação orienta o código

1. Todo cartão deve partir de requisitos e regras presentes no SERS.
2. Os cenários Gherkin devem orientar o TDD e os testes de integração/interface quando aplicáveis.
3. Alterações de modelo devem preservar o DER, o schema Prisma e migrations novas - migrations aplicadas nunca são alteradas.
4. Mudanças arquiteturais relevantes devem atualizar o DAS, os diagramas e o planejamento correspondente.
5. A documentação deve distinguir claramente o que está implementado do que ainda é planejado.

## Processo de desenvolvimento

- O trabalho ocorre em branches `feature/*`.
- O fluxo usual é `feature/*` -> `develop` -> `main`, por pull request e CI verde.
- O Definition of Done inclui testes, lint, build, integração cliente/servidor, estados de interface, segurança e atualização documental proporcional.
- Há bancos Neon distintos para desenvolvimento, testes e deploy. Operações de migration devem sempre apontar para o ambiente correto.
- Não há commits automáticos de alterações não revisadas ou de artefatos fora do escopo do cartão.

## Repositórios relacionados

- [comanga-api](https://github.com/IsaacLeite1309/comanga-api) - API Node.js/Express, Prisma, PostgreSQL, sessão e mídia.
- [comanga-web](https://github.com/IsaacLeite1309/comanga-web) - SPA React, TypeScript, Vite e Tailwind CSS.

## Leitura recomendada

Para entender o projeto a partir do zero, leia nesta ordem:

1. SERS.
2. User Stories e cenários Gherkin.
3. DAS.
4. DER e diagrama de classes Prisma.
5. Cartões Kanban e plano geral de desenvolvimento.
