# Documento de Arquitetura de Software do CoMangá

## 1. Introdução

O CoMangá é uma plataforma web para catalogação de mangás publicados no Brasil. O domínio organiza **Obra → Edição brasileira → Volume** e oferece um catálogo público consultável e uma área administrativa de curadoria.

Implementado: cadastro, ativação, login, logout, recuperação, redefinição e alteração de senha, exclusão de conta; perfis de acesso com perfil ativo por sessão; administração de contas e de opções; cadastro, publicação e exclusão de Obras, Edições e Volumes; importação de capas por URL; catálogo público com pesquisa, filtros, detalhes, Obras por Autor e navegação entre Volumes; política de conteúdo adulto.

Planejado (requisitos aprovados no SERS, sem implementação): Estante Digital, Lista de Desejos, calendário de lançamentos e enriquecimento autenticado das respostas públicas. As telas "Coleção", "Checklist", "Desejos" e a seleção de Volumes existem apenas como protótipos de interface, sem API nem persistência.

Este documento registra as decisões arquiteturais vigentes. O detalhamento de entidades, fluxos e rotas está no [modelo textual dos diagramas](../05-Diagramas/Modelo-Textual-para-IA.md), e o estado dos diagramas visuais está no [README dos diagramas](../05-Diagramas/README.md). Requisitos e cenários de qualidade estão no [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md) e nos [cenários ATAM](../03-ATAM/Cen%C3%A1rios%20ATAM%20-%20CoManga.md).

## 2. Visão geral e implantação

O CoMangá é uma aplicação cliente-servidor com dois repositórios independentes:

- `comanga-web`: SPA em React, TypeScript e Vite, hospedada na Vercel;
- `comanga-api`: API REST em Node.js, Express e TypeScript, publicada na Render como um único processo (monólito modular).

Fluxo principal:

```text
Navegador -> SPA na Vercel -> (Vercel reescreve /api/*) -> API na Render -> Prisma -> PostgreSQL no Neon
                                                             API -> Cloudflare R2 (capas)
                                                             API -> Resend (e-mails de conta)
Navegador -> origem pública de mídia (MEDIA_PUBLIC_BASE_URL) para carregar as capas
```

O `vercel.json` da Web reescreve `/api/:path*` para `https://comanga-api.onrender.com/api/:path*` e as demais rotas para `index.html`. Para o navegador, a API fica na mesma origem da SPA, o que permite o cookie de sessão com `SameSite=Strict`. O diagrama de implantação em Mermaid está no modelo textual.

## 3. Representação arquitetural

### 3.1. Web (`comanga-web`)

A SPA usa React Router para as rotas, Axios para HTTP (`baseURL` = `VITE_API_URL` ou `/api`, sempre com credenciais), Tailwind CSS para a interface, Sonner para notificações e Vitest com React Testing Library para testes. Em desenvolvimento, o Vite serve na porta 8080 e encaminha `/api` para `VITE_API_PROXY_TARGET` (padrão `http://localhost:3000`).

O código é dividido em três categorias:

| Categoria | Pastas | Papel |
| --- | --- | --- |
| Composição | `src/App.tsx`, `src/main.tsx`, `src/app/` | Rotas, guardas de rota (`GuestRoute`, `ProtectedRoute`, `PublicProfileRoute`), navegação e página 404. |
| Features | `src/features/<nome>/` | `auth`, `profile`, `admin-users`, `admin-catalog`, `admin-media`, `public-catalog`, `collection`, `wishlist`. |
| Compartilhado | `src/components`, `src/hooks`, `src/lib`, `src/services`, `src/test`, `src/vite-env.d.ts`, `src/index.css` | Componentes de interface, cliente HTTP e utilitários. |

Dependências permitidas entre features: `profile` e `admin-users` usam `auth`; `admin-catalog` usa `admin-media`; as demais não dependem de outras features.

Guardas de rota (a API revalida tudo; as guardas só orientam a navegação):

- rotas de visitante (`/entrar`, `/cadastrar`, `/reenvio`, `/recuperar-senha`, `/redefinir-senha/:token?`) enviam uma conta autenticada ao perfil;
- rotas de perfil exigem sessão;
- rotas públicas do catálogo enviam ao perfil, com o aviso "Mude o perfil para usuário padrão para acessar essa página.", a conta cujo perfil ativo é Administrador;
- rotas `/admin/*` exigem o perfil ativo Administrador e são carregadas sob demanda (`lazy`).

As URLs públicas de Edição e Volume são sempre contextualizadas (`/obras/:slug/edicao/:editionId` e `/obras/:slug/edicao/:editionId/volume/:volumeId`). A tabela completa de rotas está no modelo textual.

### 3.2. API (`comanga-api`)

A API concentra regras de negócio, validação (Zod), autenticação, autorização, visibilidade, política de conteúdo adulto, tratamento de erros e integração com serviços externos. É um monólito modular: um único deploy, com fronteiras internas explícitas e verificadas por teste.

| Categoria | Pastas | Papel |
| --- | --- | --- |
| Composição | `src/app.ts`, `src/server.ts`, `src/routes/`, `src/middlewares/`, `src/commands/` | Monta o Express, registra as rotas, aplica middlewares e expõe comandos de manutenção. |
| Módulos de domínio | `src/modules/` | Regras de negócio por domínio (tabela abaixo). |
| Compartilhado | `src/infrastructure/`, `src/prisma.ts`, `src/database/`, `src/errors/`, `src/utils/`, `src/types/` | Adaptadores de infraestrutura, cliente Prisma, erros e utilitários. |

| Módulo | Responsabilidade | Depende de |
| --- | --- | --- |
| `auth` | Cadastro, ativação, login, logout, recuperação e redefinição de senha, sessão, perfis de acesso, `requireRole`, limitadores de tentativa | — |
| `users` | Conta própria: dados, preferência +18, perfil ativo, nome de usuário, senha e exclusão | `auth` |
| `admin/users` | Listagem de contas e concessão/remoção do perfil Administrador | `auth` |
| `admin/options` | "Gerenciar opções" (listas gerenciáveis como autores, editoras, acabamentos, formatos e miolos) e opções dos formulários de Obra e Edição. Tipos de obra e gêneros são valores fixos do sistema, sem gestão pelo Administrador. | `catalog` |
| `catalog` | Obras, Edições e Volumes na administração, visibilidade e integridade | `media` |
| `media` | Importação, associação, descarte e limpeza de capas | — |
| `public-catalog` | Catálogo público, detalhes, filtros e política de conteúdo adulto | — |

A infraestrutura fica em `src/infrastructure`:

- `contracts/`: interfaces `MailService`, `MediaStorage` e `RateLimitStore`, das quais os módulos dependem;
- `container.ts`: instancia os adaptadores de e-mail e mídia (`ResendMailService`, `R2MediaStorage`, `downloadRemoteImage`, `processCoverImage` e o resolvedor de URL pública). Os limitadores de tentativa criam o armazenamento em memória no próprio módulo `auth`, e os mapeadores do catálogo importam o resolvedor de URL pública diretamente. O R2 é instanciado sob demanda, então a API sobe sem credenciais de mídia e só falha ao usar a mídia;
- `mail/`, `media/`, `rate-limit/`, `logging/`, `operations/` e `database/`: adaptadores e operação.

O fluxo usual de uma requisição é: contexto da requisição (identificador) → log → CORS → JSON → rota → middleware de sessão e de perfil → validação Zod → regra do módulo → Prisma (em transação quando há várias escritas) → mapeamento da resposta → tratador de erros. A adoção de portas e adaptadores é pragmática: os contratos isolam e-mail, armazenamento de mídia e rate limiting, mas os módulos acessam o Prisma diretamente.

Rotas montadas: `/health`, `/api/auth`, `/api/users`, `/api/admin` e `/api/public`, além de `/ping` (verificação simples do banco).

### 3.3. Regras de modularidade verificadas

As duas aplicações carregam a política `comanga/architecture` no `eslint.config.js` (plugins `eslint-plugin-boundaries` e `eslint-plugin-import-x`) e a validam com `npm run test:architecture` (`scripts/eslint-architecture.test.cjs`). O teste monta projetos temporários e confirma que o lint:

- aceita que um módulo/feature importe outro apenas pela entrada pública `index.ts` (inclusive pelo alias `@/`) e apenas na direção declarada;
- recusa imports de arquivos internos de outro módulo por `import`, reexportação, import dinâmico e imports só de tipo;
- recusa dependência invertida (o provedor importar o consumidor);
- permite que a composição importe as entradas públicas, mas impede que um módulo use a composição como ponte;
- impede que o código compartilhado importe ou reexporte módulos de domínio;
- recusa arquivos fora das categorias conhecidas, mesmo usados como ponte;
- impede que código de produção importe arquivos de teste;
- detecta ciclos de importação em tempo de execução (ignorando ciclos só de tipos);
- recusa `require`, `module.require`, `require.resolve`, `createRequire`, referências `/// <reference path>` e import dinâmico com caminho calculado;
- recusa imports não resolvidos;
- na API, garante que o comando `media:cleanup` use a entrada pública de `media`;
- confirma que a configuração real do repositório bloqueia um acesso interno.

O mesmo teste fixa os limites de manutenibilidade como erro: complexidade ciclomática até 15, profundidade de blocos até 4 e tamanho de função até 80 linhas de código na API e 150 na Web. Testes e arquivos gerados ficam isentos apenas do limite de tamanho.

## 4. Persistência

O PostgreSQL é hospedado no Neon, com bancos separados para desenvolvimento, testes e deploy (informação confirmada pelo responsável pelo projeto; o provedor não aparece no código). Só a API acessa o banco, pelo Prisma ORM. A Web nunca se conecta ao banco.

O modelo cobre contas (usuários, perfis, perfis concedidos, sessões, tokens de redefinição), o catálogo (Obras, Edições, Volumes e seus vínculos, incluindo `EditionPaper` para os miolos ordenados da Edição), opções administrativas com dependências e metadados de mídia. As entidades e cardinalidades estão no modelo textual.

Decisões de persistência:

- **Migrations versionadas** em `prisma/migrations` são parte obrigatória da implantação. Migration aplicada não é alterada; correções entram em migration nova. `npm run migrate` aplica as migrations em ambientes já existentes; `npm run migrate:dev` cria migrations no desenvolvimento; `npm run migrate:test` aplica no banco de teste.
- **Integridade no banco**: chaves estrangeiras `RESTRICT` na hierarquia Obra → Edição → Volume, nas capas e nos valores de opção; `CASCADE` em sessões, perfis concedidos, tokens de redefinição, vínculos da Obra e variantes de mídia. Constraints `CHECK` limitam valores fechados (status, visibilidade, moedas, estados de mídia).
- **Gatilhos**: validam e travam o ciclo de vida das capas (uma capa ativa pertence a um único registro; capa desassociada passa a `Descartando`) e, quando uma conta é criada ou a coluna legada `nivel_acesso` muda, recriam a partir dela os perfis concedidos e o perfil preferido (é assim que a conta nova recebe Usuário Padrão). Não há gatilho no sentido inverso: a API grava as duas coisas, e uma escrita direta em `user_profiles` não atualiza `nivel_acesso`.
- **Transações e bloqueios**: escritas com várias tabelas rodam em transação Prisma. Alterações de Obras e de Volumes, exclusão de Volumes e mudanças de visibilidade de Obras e Edições usam um bloqueio consultivo global do catálogo, o mesmo dos gatilhos de mídia; operações de conta usam bloqueio de linha do usuário; concessão e remoção do perfil Administrador usam um bloqueio consultivo próprio para proteger o último Administrador.
- **Índices**: trigramas (`pg_trgm`) para busca parcial por título, título original, título romanizado, autor, nome de usuário e e-mail; índices compostos para os filtros públicos e para datas de lançamento.
- **Pool**: o Prisma usa até 5 conexões por padrão (`PRISMA_CONNECTION_LIMIT`, `PRISMA_POOL_TIMEOUT_SECONDS`). O pool `pg` de `src/database` atende os testes de integração.

Não há rotina de backup versionada no projeto; a recuperação depende dos recursos do plano do Neon (ver RNF11 nos cenários ATAM).

## 5. Mídia

O armazenamento ativo de capas é o **Cloudflare R2**, acessado pelo SDK S3 (`@aws-sdk/client-s3`) por trás do contrato `MediaStorage`. O PostgreSQL guarda apenas metadados e chaves (`media_assets`, `media_variants`), nunca binários. Cloudinary foi somente uma prova de conceito abandonada.

- **Entrada**: não há upload de arquivo. O Administrador informa uma URL HTTPS e a API faz a importação remota, genérica e sem regra por provedor.
- **Proteção da importação**: recusa URL sem HTTPS, com credenciais ou acima de 2048 caracteres; recusa `localhost` e endereços privados, reservados ou locais após resolver o DNS; fixa o IP resolvido na conexão; segue até 3 redirecionamentos revalidando cada destino; aceita apenas JPEG, PNG, WebP e AVIF; aplica limites de bytes, tempo e pixels (`MEDIA_MAX_SOURCE_BYTES`, `MEDIA_DOWNLOAD_TIMEOUT_MS`, `MEDIA_MAX_SOURCE_PIXELS`, com padrões de 10 MiB, 10 s por requisição e 40 milhões de pixels). Limitação conhecida: a checagem de endereços não reconhece prefixos IPv6 que embutem IPv4 (`64:ff9b::/96` e `2002::/16`), o que deixa uma brecha frente à regra de recusar destinos privados ou reservados.
- **Processamento**: Sharp gera um original de 1200x1800 e variantes de 640x960 e 320x480 em WebP, gravados em `covers/<id>/` com cache público imutável. Se uma gravação falhar, a API tenta remover os objetos já enviados. Essa compensação não é atômica com o banco: se a persistência e a remoção falharem, objetos podem ficar órfãos no R2, sem registro no banco. O erro da remoção é ignorado por `CoverImportService`, e `media:cleanup` não alcança objetos sem registro; ele só processa ativos em `Descartando`.
- **Associação**: o ativo nasce `Pendente` e passa a `Ativo` quando uma Obra ou um Volume o associa, na mesma transação da escrita. A Edição não tem capa própria: exibe a capa do Volume 1.
- **Substituição e limpeza**: a capa substituída passa a `Descartando` e é removida do R2 e do banco; falhas ficam para `npm run media:cleanup`. Uma capa importada e não associada pode ser descartada pelo próprio Administrador.
- **Entrega**: a API devolve URLs montadas a partir de `MEDIA_PUBLIC_BASE_URL` (HTTPS obrigatório) e da chave do objeto; a URL de origem nunca é exibida. O navegador carrega as imagens direto da origem pública de mídia.

## 6. E-mail

Os e-mails de ativação e de redefinição de senha são enviados pela API HTTP do **Resend** (`RESEND_API_KEY`, `RESEND_FROM`), atrás do contrato `MailService`. O link usa `FRONTEND_URL` (ou a primeira origem de `CORS_ORIGIN`).

- Cada envio tem limite de 10 segundos. Não há fila persistente nem nova tentativa automática.
- No cadastro, falha de envio mantém a conta criada e orienta o reenvio; no reenvio, a falha responde 502; na recuperação de senha, a resposta ao usuário já foi enviada (neutra) e a falha só é registrada em log.
- Respostas do provedor não são repassadas ao cliente nem ao log, para não expor destinatários.

## 7. Autenticação, sessão e autorização

### 7.1. Sessão stateful

O CoMangá não usa JWT. No login, a API compara a senha com bcrypt (custo 10), gera um token aleatório de 48 bytes, grava apenas o hash SHA-256 em `sessions` e entrega o token em cookie HttpOnly (`SESSION_COOKIE_NAME`, padrão `comanga_session`), com `SameSite=Strict` por padrão (`COOKIE_SAME_SITE`) e `Secure` em produção ou atrás de HTTPS (`COOKIE_SECURE` força o valor). O token não é exposto ao JavaScript.

Em cada rota protegida, o middleware calcula o hash do cookie, exige sessão não revogada e conta Ativada, recarrega os perfis concedidos e atualiza o último uso no máximo a cada 5 minutos (`SESSION_TOUCH_INTERVAL_MS`). A sessão é revogada no logout; a redefinição de senha revoga todas as sessões da conta; a alteração autenticada de senha revoga as demais; a exclusão da conta remove as sessões. Não há expiração por tempo: a sessão vale até ser revogada, e o cookie não define validade (é descartado quando o navegador encerra a sessão).

### 7.2. Perfis e autorização

A autorização usa perfis, não uma role única:

- **perfis concedidos** (`user_profiles`): toda conta tem Usuário Padrão; o Administrador é concedido ou removido por outro Administrador, nunca pela própria conta, e o último Administrador ativo é protegido;
- **perfil preferido** (`users.preferred_profile_id`): define o perfil ativo inicial do próximo login, se ainda concedido;
- **perfil ativo** (`sessions.active_profile_id`): contexto da sessão, trocado pelo próprio usuário entre os perfis concedidos, sem conceder nem remover perfis;
- **nível de acesso** (`users.nivel_acesso`): coluna legada; um gatilho recria os perfis concedidos a partir dela; não decide autorização.

O middleware `requireRole('Administrador')` protege todas as rotas `/api/admin/*` e só autoriza quando o perfil ativo é Administrador **e** a conta ainda tem essa atribuição. Sem isso, responde 403 antes de qualquer regra de negócio. Se a atribuição for removida, a sessão passa a operar como Usuário Padrão na requisição seguinte.

### 7.3. Catálogo público e conteúdo adulto

As rotas `/api/public/*` usam sessão opcional: sem cookie, com cookie inválido ou com conta não Ativada, a requisição é tratada como visitante. Registros privados nunca são retornados, inclusive por URL direta. A política de conteúdo adulto (`adultContentPolicy.ts`) é única para todas as consultas públicas:

- visitante e menor de 18 anos não veem conteúdo adulto;
- conta sem o perfil Administrador vê conteúdo adulto somente se for maior de idade e tiver a preferência +18 ativa;
- conta com o perfil Administrador concedido lê conteúdo adulto independentemente de idade, preferência e perfil ativo (só leitura);
- sem autorização, a Obra é ocultada pela flag adulta e também pela associação a Hentai ou a gênero exclusivo adulto.

Respostas públicas que dependem do visitante usam `Cache-Control: private, no-store` e `Vary: Cookie`.

### 7.4. Rate limiting e CORS

Os limitadores guardam estado na memória do processo (`MemoryRateLimitStore` ou mapa interno), por janela fixa contada a partir da primeira ocorrência:

| Alvo | Chave | Limite | Resposta |
| --- | --- | --- | --- |
| Login | IP | 5 falhas (HTTP 401) em 5 minutos; login bem-sucedido zera a contagem | 429 `LOGIN_RATE_LIMITED`, sem consultar o banco |
| Recuperação e redefinição de senha | IP | 5 requisições em 15 minutos, por rota | 429 `RECOVERY_RATE_LIMITED` com `Retry-After: 900` |
| Alteração de senha autenticada | Conta | 5 falhas em 5 minutos | 429 `PASSWORD_CHANGE_RATE_LIMITED` |
| Emissão de token de redefinição | Conta | 1 token a cada 60 segundos | Resposta neutra, sem novo e-mail |

Consequências e lacunas:

- a contagem recomeça quando o processo reinicia e não seria compartilhada entre várias instâncias;
- o limitador de recuperação passa a recusar qualquer IP novo quando o mapa atinge 10.000 IPs dentro da janela;
- cadastro (`/register`) e reenvio de ativação (`/resend-activation`) não têm rate limiting, nem a exclusão de conta (`DELETE /api/users/me`), que confere a senha atual sem limite; o reenvio responde 404 para e-mail não cadastrado, o que permite descobrir se um e-mail tem conta;
- a API confia em um salto de proxy (`trust proxy = 1`) para obter o IP do cliente; com a reescrita da Vercel para a Render, provavelmente há dois proxies na frente da API, e `req.ip` pode ser o IP de saída da Vercel. Nesse caso, os limites por IP de login e de recuperação seriam compartilhados entre usuários. É preciso confirmar em produção.

O CORS aceita apenas as origens listadas em `CORS_ORIGIN`, com credenciais. Requisições sem cabeçalho `Origin` (GET de mesma origem e ferramentas) são aceitas. As escritas da SPA chegam com `Origin` igual ao domínio da Vercel, que precisa constar em `CORS_ORIGIN`; origens não listadas recebem 403.

## 8. Observabilidade e operação

- Logs JSON estruturados com identificador da requisição, método, rota registrada (sem query nem parâmetros), status e duração; stack trace apenas fora de produção.
- Tratador global de erros: erros 5xx respondem `{ error: "Erro interno do servidor.", code }` sem detalhes internos; códigos internos do ORM não saem na resposta; violações de chave estrangeira viram 409 `DATA_INTEGRITY_CONFLICT`. Erros em rotas de autenticação não levam o objeto de erro ao log.
- `/health/live` (processo), `/health/ready` (banco) e `/health/metrics` (CPU, memória e conexões do PostgreSQL, protegido pelo cabeçalho `x-ops-token` = `OPS_METRICS_TOKEN`; sem token configurado, a rota não existe).
- Desligamento gracioso em `SIGTERM`/`SIGINT`: fecha o HTTP e o Prisma, com tempo máximo configurável (`GRACEFUL_SHUTDOWN_TIMEOUT_MS`, padrão 10 s).
- Comandos de manutenção: `npm run check:covers` (confere capas antes de migrations de mídia) e `npm run media:cleanup` (processa até 20 capas em `Descartando`).

## 9. Qualidade e integração contínua

Testes por camada:

- **API**: Jest com Supertest. Testes unitários (`*.unit.test.js`, `npm run test:unit`) rodam sem banco; testes de integração (`npm test`) exigem um banco exclusivo cujo nome seja `test`, comece com `test_` ou termine em `_test` e seja diferente de `DATABASE_URL` e `DIRECT_URL`. Há testes de planos de consulta (índices) e de migrations.
- **Web**: Vitest com React Testing Library para componentes, páginas, contexto de autenticação e guardas de rota.
- **Arquitetura**: `npm run test:architecture` nas duas aplicações (seção 3.3).
- **Cobertura**: limite global de 80% em linhas, instruções, ramos e funções nas duas aplicações.
- **Capacidade**: `npm run load:test` na API mede P95, RPS, taxa de 5xx, tamanho e itens por resposta contra um ambiente autorizado (ver `comanga-api/docs/operations/capacity-test.md`).

Validação: `npm run check` é executado localmente, de forma manual.

- API: lint, arquitetura, build TypeScript e testes unitários com cobertura. `npm run check:integration` roda lint, arquitetura, migrations no banco de teste e a suíte completa com cobertura, sem o build.
- Web: lint, arquitetura, build (typecheck + Vite) e testes com cobertura.
- `npm run check:online` executa `npm audit`.

Cada repositório tem `.github/workflows/quality.yml` (a API sobe um PostgreSQL 16 de serviço e roda `check:integration`; a Web roda `check`), mas o GitHub Actions da conta está bloqueado por cobrança. Os workflows não estão sendo executados e não servem como evidência de aprovação; a validação real é a execução local.

## 10. Ambientes e deploy

- **Web**: Vercel, com o `vercel.json` descrito na seção 2. O build é `npm run build` (typecheck + Vite). Variável: `VITE_API_URL` (opcional; sem ela a Web usa `/api`).
- **API**: Render (`comanga-api.onrender.com`). O repositório fornece `npm run build`, `npm run migrate` e `npm start`; os comandos de build e início ficam configurados na plataforma, não em arquivo versionado. O README da API orienta gerar o Prisma Client, aplicar as migrations versionadas e compilar antes de iniciar.
- **Banco**: Neon, com um banco por ambiente (desenvolvimento, testes e deploy). Migrations e comandos de manutenção devem ser direcionados conscientemente ao banco do ambiente.
- **Mídia e e-mail**: Cloudflare R2 (`R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET`, `MEDIA_PUBLIC_BASE_URL`) e Resend (`RESEND_API_KEY`, `RESEND_FROM`).
- **Demais variáveis da API**: `DATABASE_URL`, `DATABASE_URL_TEST`, `DATABASE_SSL`, `CORS_ORIGIN`, `FRONTEND_URL`, cookies e sessão (`SESSION_COOKIE_NAME`, `COOKIE_SECURE`, `COOKIE_SAME_SITE`, `SESSION_TOUCH_INTERVAL_MS`), operação (`PORT`, `NODE_ENV`, `OPS_METRICS_TOKEN`, `REQUEST_LOGGING_ENABLED`, `GRACEFUL_SHUTDOWN_TIMEOUT_MS`) e pools. A lista completa e os padrões estão no README da API.

Segredos ficam apenas nas configurações de cada plataforma, nunca no repositório.

## 11. Restrições, riscos e limitações conhecidas

- **Custo controlado**: a solução usa planos gratuitos ou de baixo custo da Vercel, da Render, do Neon e do Cloudflare R2. O plano gratuito da Render pode adormecer a API e causar partida a frio.
- **Instância única**: sessões ficam no banco, mas o rate limiting fica em memória. Escalar para várias instâncias exigiria um `RateLimitStore` compartilhado.
- **Sessão sem expiração por tempo**: a sessão só termina por revogação. A expiração é planejada no RNF17 do SERS.
- **E-mail sem fila**: falhas do provedor não são reenviadas automaticamente.
- **Capacidade não medida**: as metas de RNF01 a RNF03 têm ferramenta de medição, mas ainda não há resultado registrado; a meta de TTFB das capas (RNF04) não tem mecanismo de medição (ver cenários ATAM).
- **Integridade e privacidade**: a API é a autoridade sobre sessão, perfis, visibilidade e conteúdo adulto; as guardas da Web são apenas conveniência de navegação.
- **Evolução**: Estante Digital, Lista de Desejos, calendário e enriquecimento autenticado podem ser acrescentados como novos módulos na API e novas features na Web, sem mudar a topologia.

## 12. Componentes abandonados ou removidos

Não fazem parte da arquitetura atual:

- Cloudinary (prova de conceito substituída pelo R2; ver `comanga-api/docs/decisions/cloudinary-poc.md`);
- SMTP/Nodemailer (substituídos pela API HTTP do Resend);
- JWT ou sessão em `localStorage`;
- tipo de edição, quantidade de volumes originais e capa própria da Edição;
- gestão de tipos de obra e gêneros pelo Administrador (são valores fixos do sistema; resta na API a alteração do campo `active`, SERS 6.3);
- controle de acesso por uma única role em `nivel_acesso` (substituído por perfis concedidos e perfil ativo).
