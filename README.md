# CoMangá Docs

Este repositório concentra os artefatos de documentação de engenharia do projeto CoMangá, uma plataforma para gerenciamento e catalogação de coleções físicas de mangás.

A documentação aqui registrada serve como base para alinhar requisitos, regras de negócio, arquitetura, modelagem, critérios de aceite e implementação dos repositórios de código do sistema.

## Estrutura

- `01-SERS/`: Especificação de Requisitos de Software.
- `02-User-Stories/`: User Stories e critérios de aceite em Gherkin.
- `03-ATAM/`: Cenários de análise arquitetural baseados nos requisitos não funcionais.
- `04-DAS/`: Documento de Arquitetura de Software.
- `05-Diagramas/`: Diagramas atualizados para a etapa N2.

## Principais artefatos

### SERS

Define os requisitos funcionais, requisitos não funcionais e regras de negócio do sistema. É a principal referência para entender o comportamento esperado da aplicação e as restrições que devem ser respeitadas pelo código.

### User Stories + Gherkin

Traduz os requisitos em histórias de usuário, critérios de aceite e cenários comportamentais testáveis. Esses cenários orientam a implementação dos cartões e a criação dos testes automatizados.

### ATAM

Registra cenários de análise arquitetural baseados nos requisitos não funcionais do projeto, apoiando a avaliação de atributos de qualidade como segurança, desempenho, disponibilidade, manutenibilidade e confiabilidade.

### DAS

Descreve a arquitetura do sistema CoMangá, baseada na stack PERN: PostgreSQL, Express, React e Node.js. O documento também registra decisões arquiteturais sobre camadas, persistência, segurança, hospedagem e restrições do MVP.

### Diagramas

Inclui os artefatos visuais atualizados para a N2:

- Diagrama de Casos de Uso refinado.
- Diagrama de Sequência do fluxo de login com sessão stateful.
- DER físico do banco PostgreSQL.
- Diagrama de Classes com mapeamento ORM via Prisma.

## Relação com o código

Esta documentação acompanha os repositórios:

- `comanga-web`: frontend React.
- `comanga-api`: backend Node.js/Express.
- `comanga-docs`: documentação técnica e acadêmica.

As decisões documentadas neste repositório devem orientar a implementação das funcionalidades, os testes automatizados, as migrations Prisma, os critérios de aceite dos cartões e a evolução da modelagem do projeto.
