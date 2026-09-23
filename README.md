# CoMangá Docs

Documentação técnica e acadêmica do CoMangá, plataforma web para consultar informações estruturadas de mangás publicados no Brasil e, futuramente, organizar coleções físicas. Este repositório reúne requisitos, regras de negócio, cenários de aceite, arquitetura e diagramas. Detalhes de instalação e operação ficam nos READMEs da API e da Web.

## O sistema documentado

O CoMangá tem uma SPA React hospedada na Vercel e uma API REST Node.js/Express hospedada na Render. A API usa PostgreSQL no Neon com Prisma, sessões stateful em cookie HttpOnly, Cloudflare R2 para as capas processadas e Resend para os e-mails transacionais.

```text
SPA React (Vercel) -> API REST (Render) -> PostgreSQL (Neon)
                                         -> Cloudflare R2 (capas)
                                         -> Resend (e-mail)
```

**Implementado:** cadastro, ativação, login, logout, recuperação e alteração de senha, exclusão de conta; perfis Usuário Padrão e Administrador com perfil ativo por sessão; administração de usuários e de opções do catálogo; cadastro de Obras, Edições e Volumes; importação e armazenamento interno de capas; catálogo público com busca e filtros; detalhes públicos com navegação contextual Obra → Edição → Volume; listagem de Obras por Autor; restrição de conteúdo adulto.

**Planejado, sem implementação:** calendário público de lançamentos, Estante Digital, Lista de Desejos e o estado de posse/desejo do usuário no catálogo. A Web já tem as rotas `/colecao`, `/checklist` e `/desejos`, mas só como telas “Em breve”.

## Onde encontrar cada resposta

| Dúvida | Documento |
| --- | --- |
| O que o sistema deve fazer, regras de negócio, atores e restrições | [SERS](./01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md) |
| Como verificar um comportamento, critérios de aceite | [User Stories e Cenários Gherkin](./02-User-Stories/User%20Stories%20e%20Cen%C3%A1rios%20Gherkin%20-%20CoMang%C3%A1.md) |
| Como a solução está organizada e por quê | [Documento de Arquitetura de Software (DAS)](./04-DAS/DAS%20-%20CoManga%20-%20Atualizado.md) |
| Metas de qualidade (desempenho, segurança, disponibilidade) e como são medidas | [Cenários ATAM](./03-ATAM/Cen%C3%A1rios%20ATAM%20-%20CoManga.md) |
| Modelo de dados, casos de uso e fluxos em forma visual | [Diagramas](./05-Diagramas/README.md) e [modelo textual](./05-Diagramas/Modelo-Textual-para-IA.md) |
| Rotas, variáveis de ambiente, migrations, testes e deploy da API | [README da comanga-api](https://github.com/IsaacLeite1309/comanga-api#readme) |
| Rotas, proteções de acesso, testes e deploy da Web | [README da comanga-web](https://github.com/IsaacLeite1309/comanga-web#readme) |

## Estrutura

| Diretório | Conteúdo |
| --- | --- |
| [`01-SERS`](./01-SERS/) | Especificação de Requisitos de Software: requisitos funcionais e não funcionais, regras de negócio e restrições. |
| [`02-User-Stories`](./02-User-Stories/) | Histórias de usuário e cenários Gherkin. |
| [`03-ATAM`](./03-ATAM/) | Cenários de atributos de qualidade para análise arquitetural. |
| [`04-DAS`](./04-DAS/) | Arquitetura adotada e suas decisões. |
| [`05-Diagramas`](./05-Diagramas/) | Sequência do login stateful, casos de uso, DER físico, classes ORM Prisma e o modelo textual que os descreve. |

## Markdown e PDF

Os arquivos `.md` são a fonte de manutenção. SERS, User Stories, ATAM e DAS também têm um PDF diagramado para leitura e entrega. O PDF é gerado a partir do `.md` e pode ficar para trás. Em caso de divergência, vale o `.md`. O [README dos diagramas](./05-Diagramas/README.md) informa quais imagens precisam ser regeneradas.

## Como a documentação orienta o código

1. Toda mudança de comportamento parte de um requisito ou regra do SERS.
2. Os cenários Gherkin orientam os testes de integração e de interface.
3. Mudanças de modelo exigem migration nova. Migrations já aplicadas nunca são alteradas. O DER e o modelo textual acompanham o schema Prisma.
4. Mudanças arquiteturais atualizam o DAS e os diagramas.
5. A documentação separa o que está implementado do que é planejado ou foi cancelado.

## Processo de desenvolvimento

- O trabalho ocorre em branches `feature/*`, integradas por pull request no fluxo `feature/*` → `develop` → `main`.
- API e Web têm workflows de qualidade no GitHub Actions, mas o Actions da conta está bloqueado por cobrança. A validação efetiva é rodar `npm run check` localmente em cada repositório antes do push.
- O Definition of Done inclui testes, lint, build, integração cliente/servidor, estados de interface, segurança e atualização proporcional da documentação.
- Há bancos Neon distintos para desenvolvimento, testes e deploy. Toda operação de migration deve apontar para o ambiente correto.

## Repositórios relacionados

- [comanga-api](https://github.com/IsaacLeite1309/comanga-api): API Node.js/Express em TypeScript, Prisma, PostgreSQL, sessão, mídia e e-mail.
- [comanga-web](https://github.com/IsaacLeite1309/comanga-web): SPA React, TypeScript, Vite e Tailwind CSS.

## Leitura recomendada

Para entender o projeto do zero, leia nesta ordem:

1. SERS.
2. User Stories e cenários Gherkin.
3. DAS.
4. Diagramas, a começar pelo DER e pelo diagrama de classes Prisma.
5. READMEs da API e da Web, para instalar e operar.
