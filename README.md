# CoMangá Docs

Documentação técnica do CoMangá, aplicação de catálogo e coleção de mangás publicados oficialmente no Brasil. A versão de referência é a 0.9 — homologação.

## Documentos

- [SERS: RF, RNF e RN](<01-SERS/SERS - CoMangá - Revisado.md>)
- [Histórias de usuário e cenários Gherkin](<02-User-Stories/User Stories e Cenários Gherkin - CoMangá.md>)
- [Cenários de qualidade ATAM](<03-ATAM/Cenários ATAM - CoManga.md>)
- [Documento de Arquitetura de Software](<04-DAS/DAS - CoManga - Atualizado.md>)
- [Diagrama de arquitetura em produção](<05-Diagramas/Arquitetura da Solução em Produção - CoMangá2.1.png>)
- [DER físico](<05-Diagramas/DER Físico - CoMangá.png>)
- [Evidências da N2](07-Evidencias/)

## Sistema

O frontend React/TypeScript é hospedado na Vercel; a API Express/Node.js roda na Render. O backend usa PostgreSQL no Neon via Prisma, Cloudflare R2 para capas e Resend para mensagens de conta. A documentação deste repositório mantém o comportamento esperado, as regras aplicadas e os limites conhecidos da versão de homologação.

Repositórios: [API](https://github.com/IsaacLeite1309/comanga-api) · [Frontend](https://github.com/IsaacLeite1309/comanga-web).
