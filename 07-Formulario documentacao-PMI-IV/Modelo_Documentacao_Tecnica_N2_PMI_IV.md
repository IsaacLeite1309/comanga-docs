PMI IV • ANÁLISE E DESENVOLVIMENTO DE SISTEMAS

> DOCUMENTAÇÃO TÉCNICA
> CoMangá
> Entrega N2 • PMI IV • 2026/2

| IDENTIFICAÇÃO | PREENCHIMENTO |
| --- | --- |
| Instituição | UNICESUSC |
| Curso | Análise e Desenvolvimento de Sistemas |
| Disciplina | PMI IV |
| Docente | Fernando Silva Moraes |
| Equipe | Isaac Leite: 2210885 — 2210885@unicesusc.edu.br<br>Diogo Vieira: 2510989 — 2510989@unicesusc.edu.br<br>Higor Pessoa: 2420399 — 2420399@unicesusc.edu.br<br>Klaudenilson Alves: 2510957 — 2510957@unicesusc.edu.br |
| Repositório | Backend: [comanga-api](https://github.com/IsaacLeite1309/comanga-api)<br>Frontend: [comanga-web](https://github.com/IsaacLeite1309/comanga-web)<br>Documentação: [comanga-docs](https://github.com/IsaacLeite1309/comanga-docs) |
| Local e data | Florianópolis • 10/2026 |

### Controle do documento

| Versão | Data | Responsável | Resumo das alterações | Status |
| --- | --- | --- | --- | --- |
| 0.1 | 21/10/2026 | Isaac Leite e equipe CoMangá | Documentação técnica da versão 0.9 — homologação, com requisitos, arquitetura, instalação, testes, evidências e operação. | Em revisão |
| 0.2 | 25/11/2026 | Isaac Leite e equipe CoMangá | Atualização da documentação para identificar o sistema como versão 1.0, preservando o histórico das evidências. | Em revisão |

### Sumário (ordem do formulário)

| 01 | Identificação | 13 | Pré-requisitos |
| --- | --- | --- | --- |
| 02 | Objetivo | 14 | Instalação |
| 03 | Escopo | 15 | Configuração |
| 04 | Visão geral da solução | 16 | Execução |
| 05 | Arquitetura | 17 | Testes |
| 06 | Tecnologias utilizadas | 18 | Evidências |
| 10 | Estrutura do projeto | 19 | Deploy |
| 07 | Requisitos funcionais | 20 | Segurança |
| 08 | Requisitos não funcionais | 21 | Limitações conhecidas |
| 09 | Regras de negócio | 22 | Troubleshooting |
| 11 | Banco de dados | 23 | Histórico de versões |
| 12 | APIs e integrações | 24 | Responsáveis |

## 1. Identificação

| Campo | Preenchimento |
| --- | --- |
| Nome do projeto | CoMangá |
| Versão do sistema | 1.0 |
| Versão do documento | 0.2 (em revisão) |
| Equipe | Isaac Leite (2210885; 2210885@unicesusc.edu.br)<br>Diogo Vieira (2510989; 2510989@unicesusc.edu.br)<br>Higor Pessoa (2420399; 2420399@unicesusc.edu.br)<br>Klaudenilson Alves (2510957; 2510957@unicesusc.edu.br) |
| Repositório | Backend: [comanga-api](https://github.com/IsaacLeite1309/comanga-api)<br>Frontend: [comanga-web](https://github.com/IsaacLeite1309/comanga-web)<br>Documentação: [comanga-docs](https://github.com/IsaacLeite1309/comanga-docs) |
| Data | 25/11/2026 |

## 2. Objetivo

O CoMangá tem como objetivo facilitar a colecionadores e leitores a identificação e consulta de informações de mangás, light novels, novels, databooks e artbooks publicados no Brasil. Para isso, reúne esses dados em um catálogo web pesquisável, com navegação contextual entre obras (a propriedade intelectual original), edições (as publicações brasileiras de cada obra) e volumes (que compõem cada edição), e permite organizar a coleção pessoal, registrando os volumes possuídos, lidos e desejados.

## 3. Escopo

| Incluído no escopo do produto | Fora do escopo do produto |
| --- | --- |
| Cadastro e autenticação de usuários.<br>Catálogo de mangás, light novels, novels, databooks e artbooks publicados oficialmente no Brasil, com informações detalhadas.<br>Estante Digital para gerenciamento da coleção pessoal.<br>Lista de Desejos para organizar itens que o usuário deseja comprar.<br>Calendário mensal de lançamentos de volumes publicados no Brasil.<br>Ferramentas de busca e filtros para o catálogo.<br>Painel administrativo para curadoria e manutenção do catálogo. | Funcionalidades de rede social, como seguir usuários e enviar mensagens diretas.<br>E-commerce ou venda direta de produtos.<br>Aplicativo móvel nativo; a solução será uma aplicação web responsiva.<br>Catalogação de obras não publicadas oficialmente no Brasil.<br>Avaliações e resenhas de usuários.<br>Informações sobre edições e volumes digitais. |

## 4. Visão geral da solução

O CoMangá é uma aplicação web responsiva de catálogo e gestão de coleções de mangás, light novels, novels, databooks e artbooks publicados oficialmente no Brasil. Colecionadores e leitores podem pesquisar e filtrar obras e edições, consultar os detalhes de cada obra, edição e volume e acompanhar lançamentos mensais. Na navegação, a pessoa parte de uma obra, escolhe uma de suas edições brasileiras e acessa os volumes correspondentes; com uma conta, também organiza sua Estante Digital, marca volumes como possuídos e lidos e mantém uma Lista de Desejos. Administradores e curadores mantêm os dados do catálogo pelo painel administrativo, incluindo o cadastro de obras, edições e volumes e o controle de visibilidade. A interface aplica as regras de acesso a conteúdo adulto do projeto.

## 5. Arquitetura

O CoMangá utiliza arquitetura cliente-servidor em três camadas: apresentação, aplicação e persistência. Em produção, frontend, backend, banco de dados e serviços externos são executados em plataformas próprias e se comunicam pelas integrações descritas no diagrama.

![Figura 1 — Arquitetura da solução em produção](<../05-Diagramas/Arquitetura da Solução em Produção - CoMangá2.1.png>)

Figura 1 — Arquitetura da solução em produção. Fonte: elaboração da equipe, com base na implementação e no DAS.

O frontend consome a API por HTTPS. Em produção, a Vercel encaminha as requisições de `/api` ao backend na Render. O backend concentra as regras de negócio e acessa o PostgreSQL no Neon por meio do Prisma. O módulo de mídia envia as capas processadas ao Cloudflare R2 e registra no banco, via Prisma, as chaves e os metadados necessários para localizá-las; os dois serviços não se conectam diretamente. Os módulos de autenticação do backend acionam o envio de e-mails pela API HTTP do Resend.

No navegador, as consultas públicas de pesquisa, lançamentos, obras, edições, volumes e autores usam um cache em memória. As respostas são consideradas atuais por dois minutos; entradas sem uso são descartadas após 30 minutos. Uma consulta iniciada pode terminar após a navegação para outra página; ao retornar, o resultado é reutilizado sem uma nova espera enquanto estiver atualizado. Pesquisa e lançamentos preservam seus filtros na URL. Mudanças nos filtros geram consultas distintas; alterações de dados pela API invalidam o cache, e a troca de sessão remove os resultados anteriores. O cache não substitui a validação nem as regras de acesso do backend.

### Responsabilidades dos componentes

| Componente | Responsabilidade | Comunicação/Dependência |
| --- | --- | --- |
| Frontend (SPA React/TypeScript na Vercel) | Apresenta o catálogo, as áreas autenticadas e os formulários administrativos no navegador. | Envia requisições HTTPS à API REST; em produção, usa o encaminhamento de `/api` configurado na Vercel. |
| Backend (API REST Express/Node.js na Render) | Valida requisições, aplica regras de negócio, autenticação, autorização e visibilidade, e coordena o acesso a dados e integrações. | Recebe chamadas do frontend; usa Prisma para acessar o Neon e integra-se ao Cloudflare R2 e ao Resend. |
| Banco de dados (PostgreSQL no Neon) | Persiste os dados relacionais do catálogo, contas, sessões e demais registros da aplicação. | É acessado somente pelo backend, por meio do Prisma ORM. |
| Armazenamento de mídia (Cloudflare R2) | Armazena as imagens de capa processadas. | O backend valida e processa as imagens e as envia ao R2; o banco guarda metadados e chaves. |
| Serviço de e-mail (Resend) | Envia mensagens transacionais de conta. | O backend envia as mensagens pela API HTTP do Resend. |

### Decisões arquiteturais relevantes

| Decisão | Motivo | Impacto/Consequência |
| --- | --- | --- |
| Separar frontend e backend como aplicações independentes, integradas por API REST | Permite desenvolver e publicar a interface e a API em seus repositórios e plataformas próprias, mantendo um contrato HTTP entre elas. | Exige configurar corretamente URL, encaminhamento `/api`, CORS e cookies de sessão entre os ambientes. |
| Manter o backend como monólito modular | Os domínios são separados em módulos no mesmo serviço, sem a complexidade operacional de microserviços. | Simplifica deploy e transações entre módulos; alterações e falhas do processo afetam a aplicação backend como um todo. |
| Acessar o PostgreSQL exclusivamente pelo backend com Prisma | Centraliza regras de acesso e integridade no servidor e oferece mapeamento relacional tipado. | O frontend não acessa o banco diretamente; consultas e alterações passam pela API. |
| Armazenar capas fora do banco de dados | Evita guardar os arquivos binários no PostgreSQL e permite usar armazenamento de objetos. | O banco mantém metadados e chaves; disponibilidade das capas depende do R2 e da configuração de URLs. |
| Usar sessão stateful com cookie `HttpOnly` | Mantém a sessão revogável no servidor e evita expor o token ao JavaScript do navegador. | Requer que frontend e backend preservem a configuração de cookies e credenciais nas requisições. |
| Manter cache temporário de consultas públicas no frontend | Evita repetir a carga de páginas já visitadas e permite concluir consultas enquanto a pessoa navega. | Os resultados são separados por parâmetros, usuário e preferência de conteúdo adulto; alterações de dados invalidam o cache e mudanças de sessão o removem. |

## 6. Tecnologias utilizadas

| Tecnologia | Versão | Finalidade | Onde é utilizada |
| --- | --- | --- | --- |
| Node.js | >=22.12.0 (requisito declarado) | Ambiente de execução do backend e das ferramentas de desenvolvimento/build | API e frontend |
| TypeScript | 6.0.3 (API); 5.9.3 (frontend) | Tipagem estática e compilação do código | API e frontend |
| Express | 5.2.1 | Framework HTTP que implementa a API REST | API |
| React e React DOM | 18.3.1 | Construção e renderização da interface web | Frontend |
| React Router DOM | 7.18.3 | Navegação entre páginas da aplicação | Frontend |
| TanStack Query | 5.104.5 | Cache em memória, deduplicação e atualização das consultas do catálogo público | Frontend |
| Vite | 8.2.1 | Servidor de desenvolvimento e geração do bundle web | Frontend |
| Tailwind CSS | 3.4.19 | Estilização da interface | Frontend |
| Prisma ORM e Prisma Client | 6.19.3 | Mapeamento do banco, consultas e migrations | API e PostgreSQL |
| PostgreSQL | Versão não fixada nos repositórios; serviço gerenciado no Neon | Persistência relacional dos dados | Banco de dados |
| Jest e Supertest | Jest 30.5.1; Supertest 7.2.2 | Testes automatizados da API | Backend |
| Vitest e React Testing Library | Vitest 4.1.11; Testing Library 16.3.2 | Testes automatizados da interface e dos componentes | Frontend |
| Vercel | Serviço gerenciado | Hospedagem do frontend e encaminhamento de `/api` para o backend | Frontend/produção |
| Render | Serviço gerenciado | Hospedagem da API REST | Backend/produção |
| Cloudflare R2 | Serviço gerenciado; acesso compatível com S3 via AWS SDK 3.1113.0 | Armazenamento de capas processadas | Mídia da API |
| Resend | Serviço externo; integração HTTP pela API, sem SDK versionado no projeto | Envio de e-mails de ativação de conta e recuperação de senha | API |

As versões de bibliotecas acima são as resolvidas nos arquivos de lock dos repositórios. Para plataformas gerenciadas, a documentação não fixa uma versão do serviço; a versão do servidor PostgreSQL também não está registrada no código.

## 10. Estrutura do projeto

O CoMangá é desenvolvido em dois repositórios de código independentes, um para a API e outro para o frontend. As árvores mostram os diretórios principais existentes, sem listar arquivos individuais.

### API — `comanga-api`

```text
comanga-api/
├── src/
│   ├── modules/
│   │   ├── admin/             # administração de usuários e opções
│   │   ├── auth/              # autenticação, sessões e perfis
│   │   ├── catalog/
│   │   │   ├── works/         # cadastro, consulta, edição e visibilidade de Obras
│   │   │   ├── editions/      # operações de Edições
│   │   │   └── volumes/       # operações de Volumes
│   │   ├── media/             # importação, armazenamento e ciclo de vida de capas
│   │   ├── personal-catalog/  # consultas e operações do acervo pessoal
│   │   ├── public-catalog/    # consultas públicas do catálogo
│   │   └── users/             # perfil e configurações de conta
│   ├── routes/                # registro e composição das rotas HTTP
│   ├── middlewares/           # autenticação, autorização e tratamento de erros
│   ├── database/              # conexão e acesso ao PostgreSQL
│   ├── infrastructure/        # integrações e recursos técnicos compartilhados
│   └── utils/                 # utilitários transversais
├── prisma/                    # schema e histórico de migrations
├── scripts/                   # migrations de teste, verificações e manutenção
└── __tests__/                 # testes agrupados por domínio e responsabilidade
```

### Frontend — `comanga-web`

```text
comanga-web/
├── src/
│   ├── app/                   # composição da aplicação, rotas e proteção de páginas
│   ├── components/            # componentes visuais compartilhados
│   ├── features/              # funcionalidades organizadas por domínio
│   │   ├── admin-catalog/     # formulários e gerenciamento do catálogo
│   │   ├── admin-media/       # importação administrativa de capas
│   │   ├── admin-users/       # gerenciamento de usuários
│   │   ├── auth/              # autenticação e recuperação de acesso
│   │   ├── public-catalog/    # pesquisa e páginas públicas
│   │   ├── profile/           # perfil e configurações do usuário
│   │   ├── collection/        # telas da área de coleção
│   │   ├── personal-library/  # integração com dados do acervo pessoal
│   │   └── wishlist/          # telas da Lista de Desejos
│   ├── hooks/                 # hooks compartilhados
│   ├── lib/                   # utilitários compartilhados
│   └── services/              # comunicação com a API
├── __tests__/                 # testes com estrutura correspondente a src/
└── public/                    # recursos estáticos servidos pelo Vite
```

| Diretório/Módulo | Responsabilidade |
| --- | --- |
| `comanga-api/src/modules` | Implementa as regras e operações da API, agrupadas por domínio. Os recursos de catálogo separam Obras, Edições e Volumes; os módulos administrativo e público organizam os fluxos correspondentes. |
| `comanga-api/src/routes`, `middlewares` e `infrastructure` | Registram endpoints, aplicam comportamentos transversais às requisições e integram serviços externos e recursos técnicos. |
| `comanga-api/prisma` e `comanga-api/scripts` | Mantêm o modelo relacional, as migrations e os comandos de teste e manutenção do banco. |
| `comanga-api/__tests__` | Agrupa testes automatizados por área, como autenticação, catálogo, usuários, mídia e banco de dados. |
| `comanga-web/src/app` e `src/components` | Compõem navegação, proteção de rotas e elementos visuais reutilizados pela interface. |
| `comanga-web/src/features` | Organiza páginas, componentes, hooks e regras próprias de cada funcionalidade, como administração, autenticação, catálogo público e perfil. |
| `comanga-web/src/services`, `src/lib` e `src/hooks` | Reúnem comunicação com a API e utilitários e hooks compartilhados entre funcionalidades. |
| `comanga-web/__tests__` | Mantém os testes de interface fora do código de produção, com organização que acompanha as áreas de `src/`. |

## 7. Requisitos funcionais

| ID | Título | Descrição | Prioridade | Critérios de aceitação |
| --- | --- | --- | --- | --- |
| RF01 | Cadastrar conta | O visitante informa nome de usuário, e-mail, nascimento e senha para criar uma conta. | Alta | Dados válidos criam conta pendente e iniciam o envio do link de ativação; duplicidade ou dados inválidos são recusados. |
| RF02 | Ativar conta | O titular confirma a conta pelo link recebido por e-mail. | Alta | Token válido ativa a conta uma vez; token expirado ou reutilizado não a ativa. |
| RF03 | Reenviar ativação | O titular de conta pendente solicita outro link de ativação. | Média | Um novo link é enviado para conta elegível e o anterior perde a validade. |
| RF04 | Iniciar sessão | O titular de uma conta ativa informa e-mail e senha para acessar o sistema. | Alta | Credenciais válidas criam sessão persistida e cookie `HttpOnly`; credenciais inválidas não criam sessão. |
| RF05 | Encerrar sessão | A pessoa autenticada termina sua sessão. | Alta | O cookie é removido e a sessão revogada deixa de autorizar rotas protegidas. |
| RF06 | Recuperar ou alterar senha | O titular solicita um link para redefinir a senha ou altera a senha em seu perfil autenticado. | Média | Link válido permite definir nova senha; alteração em sessão exige a senha atual; a senha anterior deixa de autenticar. |
| RF07 | Consultar e atualizar perfil | A pessoa autenticada consulta seus dados e altera o nome de usuário. | Média | A consulta retorna somente dados da própria conta; alteração válida aparece na consulta seguinte. |
| RF08 | Selecionar perfil ativo | Uma conta com mais de um perfil escolhe qual perfil usar na sessão. | Alta | A troca atualiza o perfil ativo; menus e permissões passam a refletir o perfil escolhido, inclusive em outras abas após sincronização. |
| RF09 | Controlar conteúdo adulto | O titular altera sua preferência de exibição de conteúdo adulto. | Alta | Maior de 18 anos pode ativá-la; menor de idade não pode; o catálogo responde conforme a preferência efetiva. |
| RF10 | Excluir conta | O titular solicita a exclusão de sua conta com confirmação da senha atual. | Média | A conta, sessões e vínculos pessoais são removidos; outra conta não é afetada. |
| RF11 | Administrar usuários | O administrador consulta contas e concede ou retira a atribuição administrativa de outra conta. | Média | Busca e filtros retornam os perfis atribuídos; alteração autorizada muda o acesso da conta alvo. |
| RF12 | Manter opções administrativas | O administrador lista, cadastra, altera e exclui valores das categorias editáveis. | Alta | Novo valor aparece no formulário correspondente; valor inválido, duplicado ou em uso é recusado conforme a operação. |
| RF13 | Importar capas | O administrador importa uma imagem por URL para uso como capa interna. | Alta | A imagem é validada e armazenada na infraestrutura de mídia; falha não substitui uma capa já associada. |
| RF14 | Manter obras | O administrador cria, consulta, altera e exclui obras com autores, papéis, classificações e dados editoriais. | Alta | Uma obra válida é recuperável após o cadastro; alterações preservam campos não enviados; exclusão respeita vínculos e visibilidade. |
| RF15 | Manter edições | O administrador cria, consulta, altera e exclui edições vinculadas a uma obra. | Alta | Cada edição tem número único dentro da obra; acabamento, formato e miolos aceitam os domínios válidos; a capa exibida deriva do volume 1. |
| RF16 | Manter volumes | O administrador cria, consulta, altera e exclui volumes de uma edição. | Alta | O volume válido aparece na edição com número, capa e dados informados; número repetido na mesma edição é recusado. |
| RF17 | Controlar visibilidade | O administrador publica ou torna privadas obras e edições. | Alta | Ao publicar individualmente uma edição, o sistema exige que a obra esteja pública e que a edição tenha Volume 1 com capa válida; então publica a edição e seus volumes. Ao tornar uma obra privada, suas edições e volumes também ficam privados, desde que nenhuma edição pública tenha volumes na coleção ou lista de desejos de usuários; caso contrário, a operação é recusada. Ao publicar uma obra, o sistema verifica todas as edições e, se todas puderem ser publicadas, publica obra, edições e volumes na mesma operação; se alguma não puder, nenhuma alteração é aplicada. |
| RF18 | Explorar e pesquisar catálogo público | Visitantes e pessoas autenticadas consultam as listagens de obras e edições mesmo sem informar termo de busca e podem pesquisar por texto, filtros e ordenação. | Alta | As duas listagens abrem sem termo; os resultados contêm somente registros visíveis e compatíveis com os filtros, e uma nova pesquisa produz o conjunto correspondente. |
| RF19 | Navegar pelas fichas | O catálogo mostra páginas de obra, edição e volume com suas relações e informações públicas. | Alta | É possível seguir obra → edição → volume e voltar pelo caminho contextual; URLs de registros privados não expõem seus dados. |
| RF20 | Consultar obras por autor | A pessoa abre a página de um autor a partir da ficha de uma obra. | Média | A página lista as obras públicas associadas ao autor e permite abrir suas fichas. |
| RF21 | Consultar lançamentos | A pessoa consulta volumes previstos por mês e ano e filtra por editora ou texto. | Alta | A lista usa o período selecionado e inclui apenas volumes com mês conhecido e visibilidade pública. |
| RF22 | Manter coleção individual | A pessoa autenticada adiciona ou remove volumes da coleção e altera o estado de leitura dos volumes que estão nela. | Alta | A alteração aparece na ficha do volume e nas telas da coleção; remover um volume da coleção também remove seu estado de leitura. |
| RF23 | Alterar coleção em lote | A pessoa autenticada seleciona volumes de uma edição para incluir na coleção ou remover dela. | Alta | O lote é aplicado integralmente ou não altera nenhum vínculo; erro de validação ou gravação não deixa mudanças parciais. |
| RF24 | Manter lista de desejos | A pessoa autenticada adiciona ou remove volumes da lista de desejos. | Alta | A lista reflete a alteração; um volume que está na coleção não pode ser adicionado à lista de desejos. |
| RF25 | Consultar coleção e lista de desejos | A pessoa autenticada consulta a coleção e a lista de desejos com busca, filtros e totais. | Alta | Cada tela mostra somente os vínculos da própria conta e atualiza os totais após uma alteração. |
| RF26 | Alterar lista de desejos em lote | A pessoa autenticada seleciona volumes de uma edição para incluir na lista de desejos ou remover dela. | Média | O lote é aplicado integralmente ou não altera nenhum vínculo; erro de validação ou gravação não deixa mudanças parciais. |
| RF27 | Alterar estado de leitura em lote | A pessoa autenticada seleciona volumes da coleção de uma edição para marcar como lidos ou não lidos. | Média | O lote é aplicado integralmente ou não altera nenhum estado; volumes fora da coleção não recebem estado de leitura e erro não deixa mudanças parciais. |

### Rastreabilidade

| Requisito | Regra relacionada | Caso(s) de teste | Status |
| --- | --- | --- | --- |
| RF01 — Cadastrar conta | RN01, RN02, RN03, RN08 | CT-001, CT-026 | Implementado; Testado |
| RF02 — Ativar conta | RN03 | CT-002 | Implementado; Testado |
| RF03 — Reenviar ativação | RN03 | CT-002 | Implementado; Testado |
| RF04 — Iniciar sessão | RN04 | CT-003, CT-026 | Implementado; Testado |
| RF05 — Encerrar sessão | RN04 | CT-003 | Implementado; Testado |
| RF06 — Recuperar ou alterar senha | RN02, RN04, RN05 | CT-004 | Implementado; Testado |
| RF07 — Consultar e atualizar perfil | RN01, RN06 | CT-005 | Implementado; Testado |
| RF08 — Selecionar perfil ativo | RN06 | CT-005 | Implementado; Testado |
| RF09 — Controlar conteúdo adulto | RN06, RN08 | CT-006 | Implementado; Testado |
| RF10 — Excluir conta | RN04, RN06 | CT-007 | Implementado; Testado |
| RF11 — Administrar usuários | RN06, RN07 | CT-008 | Implementado; Testado |
| RF12 — Manter opções administrativas | RN09, RN16 | CT-009 | Implementado; Testado |
| RF13 — Importar capas | RN23, RN24 | CT-010 | Implementado; Testado |
| RF14 — Manter obras | RN09, RN10, RN15, RN16, RN23, RN24, RN25, RN26, RN27 | CT-011, CT-012, CT-027 | Implementado; Testado |
| RF15 — Manter edições | RN11, RN15, RN16, RN28, RN30 | CT-013 | Implementado; Testado |
| RF16 — Manter volumes | RN12, RN13, RN14, RN15, RN16, RN23, RN24, RN29 | CT-014 | Implementado; Testado |
| RF17 — Controlar visibilidade | RN10, RN11, RN14 | CT-015, CT-027 | Implementado; ações de 21/11 confirmadas pelo responsável; passos da proteção atual cobertos por testes automatizados. A proteção de vínculos foi coberta por testes automatizados no [PR #79 da API](https://github.com/IsaacLeite1309/comanga-api/pull/79) e na suíte [EV-007](../06-Evidencias/2026-11-25-v1/api-integracao-ci.log). |
| RF18 — Explorar e pesquisar catálogo público | RN08, RN17, RN18, RN27, RN30 | CT-016, CT-025, CT-028 | Implementado; Testado |
| RF19 — Navegar pelas fichas | RN08, RN17, RN30 | CT-017, CT-025, CT-028 | Implementado; Testado |
| RF20 — Consultar obras por autor | RN08, RN17 | CT-017 | Implementado; Testado |
| RF21 — Consultar lançamentos | RN08, RN13, RN17, RN19 | CT-018, CT-028 | Implementado; Testado |
| RF22 — Manter coleção individual | RN20, RN21 | CT-019, CT-025 | Implementado; Testado |
| RF23 — Alterar coleção em lote | RN20, RN21, RN22, RN31 | CT-020, CT-027 | Implementado; Testado |
| RF24 — Manter lista de desejos | RN20, RN21 | CT-021 | Implementado; Testado |
| RF25 — Consultar coleção e lista de desejos | RN20, RN21 | CT-022, CT-025, CT-028 | Implementado; Testado |
| RF26 — Alterar lista de desejos em lote | RN20, RN21, RN31 | CT-023 | Implementado; Testado |
| RF27 — Alterar estado de leitura em lote | RN20, RN21, RN31 | CT-024 | Implementado; Testado |

Os requisitos estão implementados e há testes automatizados para as funcionalidades. A indicação “Testado” não substitui a revalidação manual de cada CT: seus resultados e pendências estão na seção 17, e as execuções automatizadas e seus percentuais estão na seção 18.

## 8. Requisitos não funcionais

Os requisitos abaixo descrevem propriedades de qualidade e limites presentes na implementação atual. Cada um traz uma forma de verificar o comportamento descrito.

| ID | Categoria | Requisito mensurável | Como verificar |
| --- | --- | --- | --- |
| RNF01 | Paginação | Listagens paginadas da API limitam `limit` a 50 registros por página; páginas de detalhe podem retornar seus vínculos próprios sem essa paginação. | Conferir os esquemas de entrada e a recusa de `limit=51` nas listagens paginadas. |
| RNF02 | Segurança de credenciais | Senhas são persistidas com bcrypt e fator de trabalho 10; o texto da senha não aparece na resposta de perfil. | Inspecionar criação e troca de senha e o contrato de consulta da própria conta. |
| RNF03 | Segurança de sessão | O identificador opaco chega por cookie `HttpOnly`; no banco existe apenas seu hash. O cookie recebe `Secure` quando `NODE_ENV=production`, salvo configuração explícita em contrário. | Inspecionar a criação da sessão, o cabeçalho `Set-Cookie` e a configuração efetiva do ambiente. |
| RNF04 | Autorização | Rotas administrativas exigem sessão válida e perfil **ativo** de administrador; perfil apenas atribuído, mas não ativo, não basta. | Conferir os middlewares das rotas e comparar respostas de uma mesma conta nos perfis padrão e administrador. |
| RNF05 | Proteção de login | Após 5 falhas por IP em janela de 5 minutos, novas tentativas recebem HTTP 429 enquanto a janela estiver ativa. O contador é mantido na memória do processo da API. | Conferir o limitador de login e observar a resposta às tentativas consecutivas. |
| RNF06 | Integridade | Cadastro de obra, mudança de visibilidade da edição e alteração da coleção em lote usam operações transacionais; FKs, unicidade e `CHECK` do PostgreSQL protegem os vínculos e estados persistidos. | Conferir as transações dessas operações e as restrições nas migrations. |
| RNF07 | Tratamento de erro | O tratador global devolve erros em JSON, sem stack trace na resposta, e registra falhas inesperadas em log estruturado com dados da requisição. | Conferir o middleware de erro, o formato da resposta e os campos do log. |
| RNF08 | Cache e atualização | As consultas no navegador usam `staleTime` de 2 minutos e `gcTime` de 30 minutos; alterações do acervo invalidam ou atualizam as consultas relacionadas. | Conferir a configuração do cliente de consultas e os pontos de invalidação ou atualização após mutações. |
| RNF09 | Responsividade | A navegação principal usa barra inferior abaixo de 768 px e navegação lateral a partir de 768 px. | Conferir os pontos de quebra e os destinos oferecidos nas duas apresentações. |
| RNF10 | Capas e layout | O componente de capa reserva proporção 2:3 e mostra placeholder quando a mídia falha; o banco guarda chaves e metadados das capas, sem seus arquivos binários. | Conferir o componente de capa, seu tratamento de erro e o esquema de mídia. |
| RNF11 | Estados de interface | Pesquisa, lançamentos, coleção e lista de desejos apresentam estados próprios de carregamento e de resultado vazio. | Conferir os componentes dessas quatro páginas nos dois estados. |

## 9. Regras de negócio

As regras abaixo especificam **quando** as capacidades da seção 7 são permitidas, recusadas ou transformadas. Mensagens literais de interface ficam fora desta tabela porque podem mudar sem alterar a regra.

| ID | Título | Regra | Requisitos relacionados |
| --- | --- | --- | --- |
| RN01 | Identidade única | Nome de usuário aceita de 3 a 20 caracteres ASCII entre letras, números e sublinhado e deve ser único; e-mail também deve ser único entre as contas, inclusive pendentes. | RF01, RF07 |
| RN02 | Senha válida | Criação e troca de senha exigem pelo menos 8 caracteres, incluindo maiúscula, minúscula, número e caractere especial, até 72 bytes em UTF-8; confirmação divergente impede a operação. | RF01, RF06 |
| RN03 | Ciclo de ativação | Conta recém-criada começa pendente e recebe o perfil padrão. O link de ativação expira em 24 horas, só pode ser usado uma vez e um reenvio para conta pendente invalida o link anterior; contas já ativadas não recebem novo link. | RF01, RF02, RF03 |
| RN04 | Acesso por sessão | Somente conta ativa com credenciais válidas recebe sessão. Encerrar sessão, alterar ou redefinir a senha e excluir a conta invalidam as sessões aplicáveis. | RF04, RF05, RF06, RF10 |
| RN05 | Recuperação sem revelação | Solicitação de recuperação não informa se o e-mail existe. Token é gerado somente para conta elegível, expira após 1 hora, é de uso único e sua substituição invalida o anterior. | RF06 |
| RN06 | Titularidade e perfis | Operações do próprio perfil usam a identidade da sessão; ações administrativas exigem perfil de administrador ativo e ainda atribuído à conta. | RF07, RF08, RF09, RF10, RF11 |
| RN07 | Atribuição administrativa | Administrador não pode alterar sua própria atribuição; o perfil padrão permanece na conta e o último administrador efetivo não pode ser removido. | RF11 |
| RN08 | Conteúdo adulto | Preferência adulta inicia desativada e só pode ser ativada após cálculo de 18 anos completos pela data de nascimento; visitantes e contas sem preferência válida não recebem obras adultas nas consultas públicas. | RF01, RF09, RF18, RF19, RF20, RF21 |
| RN09 | Domínios da obra | Tipo, autores, gêneros, editoras e revistas usam opções administrativas compatíveis com a categoria; demografia, status e papéis usam valores controlados. Um autor aparece uma vez por obra; para ele, “História e Arte” não se combina com “História” ou “Arte”, nem estes dois papéis entre si. | RF12, RF14 |
| RN10 | Identificação da obra | A combinação de título nacional e tipo não pode duplicar a de outra obra, ignorando diferenças de caixa; nova obra inicia privada. Título original é opcional no contrato atual. | RF14, RF17 |
| RN11 | Edição e capa derivada | Uma nova edição inicia privada, mesmo quando pertence a uma obra pública. A edição não possui capa própria: sua capa é derivada do Volume 1. Para publicar uma edição individualmente, a obra matriz deve estar pública e a edição deve ter Volume 1 com capa válida. Cadastrar o Volume 1 não publica a edição automaticamente. | RF15, RF17 |
| RN12 | Identificação do volume | O número do volume é único dentro da edição e pode ser zero. Um volume marcado como único não pode coexistir com outros volumes nessa edição enquanto a marcação permanecer. | RF16 |
| RN13 | Data e ISBN do volume | A data respeita a precisão informada (dia/mês/ano, mês/ano ou ano); ISBN-10 e ISBN-13, quando fornecidos, devem passar pela validação de dígito verificador. | RF16, RF21 |
| RN14 | Visibilidade da obra, das edições e dos volumes | Ao tornar uma obra privada, o sistema torna privadas todas as suas edições e volumes, salvo quando uma edição pública possui volume na coleção ou lista de desejos de usuários: nesse caso, recusa a operação. A mudança individual de uma edição pública para privada também é recusada se houver esses vínculos. Ao publicar uma obra, o sistema verifica se todas as edições têm Volume 1 com capa válida; se qualquer edição não atender à condição, a operação inteira é recusada. Se todas estiverem aptas, obra, edições e volumes são publicados juntos. Uma obra pública pode ter edições privadas; cada edição pode ser publicada individualmente quando cumprir RN11. Os volumes recebem a visibilidade da edição na criação e acompanham suas mudanças posteriores. | RF16, RF17 |
| RN15 | Exclusão protegida | Obra pública ou com edições, edição pública ou com volumes e volume público não podem ser excluídos pela administração. | RF14, RF15, RF16 |
| RN16 | Opções administrativas | Valor duplicado na categoria é recusado sem diferenciar maiúsculas; valor referenciado não pode ser excluído. | RF12, RF14, RF15, RF16 |
| RN17 | Catálogo público | Consulta ou URL pública só pode mostrar registros públicos em toda a cadeia obra → edição → volume e deve aplicar o filtro adulto antes de entregar dados. | RF18, RF19, RF20, RF21 |
| RN18 | Busca combinada | Filtros simultâneos restringem o resultado por interseção; trocar entre obras e edições preserva termos de busca compatíveis. | RF18 |
| RN19 | Calendário mensal | Volume com data conhecida apenas por ano não é atribuído a um mês arbitrário; sem período explícito, usa-se mês e ano correntes. | RF21 |
| RN20 | Privacidade da coleção e da lista de desejos | Só a conta autenticada acessa os próprios vínculos. Novas inclusões exigem volume público ligado a edição e obra públicas; vínculos preexistentes que se tornem privados ficam ocultos, sem exclusão automática. | RF22, RF23, RF24, RF25, RF26, RF27 |
| RN21 | Estado do volume na coleção ou na lista de desejos | Para cada conta e volume existe um único vínculo, que representa coleção (`Owned`) ou lista de desejos (`Wanted`). Incluir um volume na coleção remove-o da lista de desejos; um volume da coleção não pode ser incluído na lista de desejos. O volume entra na coleção como não lido, e seu estado de leitura só existe enquanto estiver nela. | RF22, RF23, RF24, RF25, RF26, RF27 |
| RN22 | Validação do lote da coleção | Todos os volumes incluídos ou removidos em uma alteração em lote da coleção devem pertencer à edição selecionada e estar disponíveis no catálogo da conta. A operação é atômica: erro em qualquer volume preserva os vínculos anteriores do lote inteiro. | RF23 |
| RN23 | Importação segura de capa | Apenas administrador pode importar por HTTPS; origem local/privada, conteúdo inválido ou acima dos limites configurados é recusado. A URL de origem não é usada para servir a capa. | RF13, RF14, RF16 |
| RN24 | Substituição de capa | A capa antiga permanece associada até que a nova seja validada, armazenada e vinculada com sucesso; cada obra e volume mantém sua própria capa interna. | RF13, RF14, RF16 |
| RN25 | Dados mínimos da obra | Criar obra exige título nacional, título romanizado, sinopse, tipo, status de publicação original, capa interna e pelo menos um autor com papel; título original, anos de publicação, gêneros, revistas e editoras originais podem estar ausentes. | RF14 |
| RN26 | Serialização e demografia | Ao marcar lançamento direto, a obra fica sem revistas de serialização e sem demografias. A seleção de gênero com código `boys-love` ou `girls-love` também limpa as demografias da obra. | RF14 |
| RN27 | Período e classificação da obra | Quando os dois anos de publicação original são informados, o fim não pode preceder o início. Obra vinculada a gênero com código `hentai` ou marcado como exclusivo para adultos é classificada como adulta mesmo que o formulário envie `adultContent=false`. | RF14, RF18 |
| RN28 | Dados mínimos da edição | Criar edição exige obra existente, editora brasileira, número cronológico positivo e status de publicação no Brasil. Acabamento e formato aceitam ausência; a lista de miolos pode estar vazia, mas não pode repetir um valor. | RF15 |
| RN29 | Dados mínimos do volume | Criar volume exige edição existente, número inteiro não negativo, capa interna e data compatível com sua precisão; páginas, preço, ISBN, link de afiliado e sinopse são opcionais. Se marcado como volume único, seu número é fixado em 1. | RF16 |
| RN30 | Total derivado de volumes | O total exibido de volumes da edição é calculado a partir dos volumes vinculados, sem cadastro manual desse número; nas consultas públicas, contam-se apenas os volumes públicos. | RF15, RF18, RF19 |
| RN31 | Atomicidade das alterações em lote | Alterações em lote da coleção, da lista de desejos e do estado de leitura são atômicas por edição: todas as inclusões, remoções ou mudanças de leitura solicitadas são confirmadas juntas; se qualquer volume falhar na validação ou gravação, nenhuma alteração do lote é persistida. | RF23, RF26, RF27 |

A RN0042 do SERS impede tornar privada uma edição pública quando algum de seus volumes estiver na coleção ou lista de desejos de um usuário. A API também bloqueia tornar privada a obra quando isso afetaria uma edição pública nessas condições. A proteção foi integrada às branches develop e main da API; a disponibilidade no site depende do deploy.

## 11. Banco de dados

O CoMangá usa PostgreSQL hospedado no Neon. O backend acessa o banco por meio do Prisma ORM; o esquema físico está em `comanga-api/prisma/schema.prisma` e suas alterações são registradas nas migrations de `comanga-api/prisma/migrations`. O DER físico abaixo apresenta uma visão resumida das 20 tabelas da aplicação e das 31 relações de chave estrangeira do schema. A figura mostra campos selecionados; para os detalhes completos, consulte o schema Prisma.

![DER físico do CoMangá — visão resumida](../05-Diagramas/DER%20F%C3%ADsico%20-%20CoMang%C3%A1%20-%20atualizado.png)

Figura 2 — Diagrama entidade-relacionamento físico do CoMangá, em formato vertical. Fonte: elaboração da equipe com base no esquema Prisma e nas migrations do CoMangá.

As relações N:M são implementadas por tabelas associativas, cada uma com dois vínculos `1:N` e chave primária composta. A demografia e o papel de autoria são valores textuais controlados, sem FK para `domain_option_values`.

### Dicionário de dados

O dicionário destaca as chaves e os campos que sustentam as principais regras do sistema. `NN` significa obrigatório (`NOT NULL`); `NULL` indica campo opcional. A especificação completa de todos os campos permanece no esquema Prisma e nas migrations.

| Entidade | Campo | Tipo | Chave/Restrição | Descrição |
| --- | --- | --- | --- | --- |
| `users` | `id` | UUID | PK, NN | Identificador da conta. |
| `users` | `username`; `email` | VARCHAR(50); VARCHAR(255) | Cada campo UNIQUE, NN | Nome de usuário e e-mail de acesso. |
| `users` | `password_hash`; `status` | VARCHAR(255); VARCHAR(20) | NN; `status` com CHECK | Hash da senha e estado da conta. |
| `users` | `preferred_profile_id` | INTEGER | FK → `profiles.id`, NULL, `ON DELETE SET NULL` | Perfil preferido. |
| `profiles` | `id`; `code`; `name` | INTEGER; VARCHAR(40); VARCHAR(50) | `id` PK; `code` e `name` UNIQUE, NN | Perfis disponíveis. |
| `user_profiles` | `user_id`; `profile_id` | UUID; INTEGER | PK composta e FKs NN → `users.id`/`profiles.id` | Perfis concedidos a cada usuário, sem duplicação do par. |
| `sessions` | `id`; `user_id` | INTEGER; UUID | `id` PK; `user_id` FK NN → `users.id`, `ON DELETE CASCADE` | Sessão e seu titular. |
| `sessions` | `session_token_hash`; `active_profile_id` | VARCHAR(64); INTEGER | Hash UNIQUE, NN; perfil FK → `profiles.id`, NULL | Token persistido apenas como hash e perfil ativo da sessão. |
| `password_reset_tokens` | `id`; `user_id`; `token_hash` | UUID; UUID; VARCHAR(64) | `id` PK; `user_id` FK NN; hash UNIQUE, NN | Solicitação de recuperação vinculada à conta. |
| `password_reset_tokens` | `expires_at`; `used_at` | TIMESTAMPTZ | Expiração NN; uso NULL | Prazo de validade e registro de consumo do token. |
| `works` | `id`; `slug` | INTEGER; VARCHAR(255) | `id` PK; `slug` UNIQUE, NN | Identificador e endereço estável da obra. |
| `works` | `title`; `romanized_title`; `synopsis` | VARCHAR(255); VARCHAR(255); TEXT | NN | Identificação e resumo da obra. `original_title` aceita NULL. |
| `works` | `type_id`; `cover_asset_id` | INTEGER; UUID | FKs NN; capa UNIQUE → `media_assets.id` | Tipo da obra e sua capa própria. |
| `works` | `original_publication_status`; `visibility` | VARCHAR(40); VARCHAR(20) | NN, CHECK de valores permitidos | Situação original e visibilidade. |
| `editions` | `id`; `work_id`; `chronological_number` | INTEGER | `id` PK; `work_id` FK NN → `works.id`; par (`work_id`, `chronological_number`) UNIQUE | Edição numerada dentro da obra. |
| `editions` | `brazilian_publisher_id` | INTEGER | FK NN → `domain_option_values.id` | Editora brasileira. |
| `editions` | `cover_type_id`; `format_id` | INTEGER | FKs → `domain_option_values.id`, NULL | Acabamento e formato opcionais. |
| `editions` | `brazil_publication_status`; `visibility` | VARCHAR(30); VARCHAR(20) | NN, CHECK de valores permitidos | Situação brasileira e visibilidade. |
| `volumes` | `id`; `edition_id`; `number` | INTEGER | `id` PK; `edition_id` FK NN → `editions.id`; par (`edition_id`, `number`) UNIQUE | Volume numerado dentro da edição. |
| `volumes` | `cover_asset_id` | UUID | FK NN, UNIQUE → `media_assets.id` | Capa própria do volume. |
| `volumes` | `release_date_precision`; `release_year/month/day` | VARCHAR(20); INTEGER | Precisão NN; partes da data NULL conforme a precisão, com CHECK | Data de lançamento com precisão completa, mês/ano ou somente ano. |
| `volumes` | `price`; `isbn_10`; `isbn_13`; `affiliate_link` | DECIMAL(10,2); VARCHAR(20); VARCHAR(20); VARCHAR(2048) | NULL | Dados comerciais e editoriais opcionais. |
| `user_volumes` | `user_id`; `volume_id` | UUID; INTEGER | PK composta, FKs NN → `users.id`/`volumes.id`, `ON DELETE CASCADE` | Um vínculo por par usuário/volume. |
| `user_volumes` | `state`; `read` | VARCHAR(10); BOOLEAN | Estado NN; leitura NULL sob CHECK | `Owned` exige leitura definida; `Wanted` exige leitura nula. |
| `domain_option_categories` | `id`; `slug` | INTEGER; VARCHAR(80) | `id` PK; `slug` UNIQUE, NN | Categoria de opção administrativa. |
| `domain_option_values` | `id`; `category_id`; `label`; `code` | INTEGER; INTEGER; VARCHAR(120); VARCHAR(80) | `id` PK; categoria FK NN; rótulo NN; `code` NULL | Valor de opção, código e categoria. |
| `domain_option_value_dependencies` | `dependent_value_id`; `depends_on_value_id` | INTEGER | PK composta, duas FKs NN → `domain_option_values.id` | Dependência entre duas opções. |
| `work_authors` | `work_id`; `author_id` | INTEGER | PK composta, FKs NN → `works.id`/`domain_option_values.id` | Autor associado à obra. |
| `work_author_roles` | `work_id`; `author_id`; `role` | INTEGER; INTEGER; VARCHAR(40) | PK composta; FK composta NN → `work_authors`; papel com CHECK | Um autor pode ter vários papéis na mesma obra. |
| `work_genres`; `work_original_publishers`; `work_serialization_magazines` | Pares `work_id` + identificador da opção | INTEGER | PKs compostas, FKs NN → `works.id`/`domain_option_values.id` | Gêneros, editoras originais e revistas de uma obra. |
| `work_demographies` | `work_id`; `demography` | INTEGER; VARCHAR(40) | PK composta; obra FK NN; demografia com CHECK | Demografias da obra. |
| `edition_papers` | `edition_id`; `paper_id` | INTEGER | PK composta, FKs NN → `editions.id`/`domain_option_values.id` | Um ou mais tipos de papel por edição. |
| `media_assets` | `id`; `object_key`; `status` | UUID; VARCHAR(512); VARCHAR(20) | `id` PK; chave UNIQUE, NN; estado com CHECK | Metadados e chave do arquivo de capa no Cloudflare R2. |
| `media_variants` | `id`; `media_asset_id`; `kind`; `object_key` | UUID; UUID; VARCHAR(40); VARCHAR(512) | `id` PK; ativo FK NN; (`media_asset_id`, `kind`) UNIQUE; chave UNIQUE | Versões processadas de uma capa. |

### Regras de integridade relevantes

- A unicidade de `editions(work_id, chronological_number)` impede números de edição repetidos na mesma obra; `volumes(edition_id, number)` faz o mesmo para volumes na mesma edição. O número do volume aceita `0`, e a marcação de volume único é guardada em `volumes.single_volume`.
- Cada obra e cada volume possuem uma capa vinculada a `media_assets`. A edição não possui `cover_asset_id`: sua capa é obtida do volume de número `1` da própria edição. A integridade do ciclo de vida das capas também usa gatilhos nas migrations; os binários ficam no Cloudflare R2, enquanto o PostgreSQL guarda chaves e metadados.
- As opções pertencem a uma categoria. O índice `domain_option_values_category_label_unique` impede rótulos iguais dentro da mesma categoria sem diferenciar maiúsculas de minúsculas; o par (`category_id`, `code`) também é único quando há código. FKs para opções usadas pelo catálogo restringem sua exclusão enquanto houver vínculos.
- A PK composta de `user_volumes` evita duplicar um mesmo volume no acervo da pessoa. A restrição `user_volumes_state_read_check` exige `read` não nulo em `Owned` e nulo em `Wanted`; a exclusão da conta ou do volume remove esse vínculo em cascata.
- A precisão da data do volume determina quais partes da data precisam existir. A restrição vigente foi criada como `NOT VALID` para preservar linhas legadas; ainda assim, aplica-se a novos registros e a linhas alteradas. Mês e dia também têm limites por CHECK.
- Os campos `visibility` e situações de publicação têm conjuntos permitidos por CHECK. As regras de publicação conjunta de obra, edição e volume são aplicadas pelo backend, inclusive a exigência de volume 1 com capa para tornar uma edição pública.

## 12. APIs e integrações

O backend oferece uma API REST em JSON, sem prefixo de versão. Em desenvolvimento, a base é `http://localhost:3000`; as rotas funcionais ficam sob `/api`, exceto saúde e diagnóstico. A interface envia cookies com credenciais; não há autenticação Bearer. Os endpoints públicos aceitam consulta anônima e, quando há sessão válida, usam o contexto da conta para aplicar a política de conteúdo adulto e devolver o estado pessoal pertinente.

| Área | Método e rota | Autenticação | Parâmetros e resultado principal |
| --- | --- | --- | --- |
| Operação | `GET /health/live`, `GET /health/ready`, `GET /ping` | Nenhuma | Confirma processo ativo; prontidão consulta o PostgreSQL. `/ping` também retorna o horário do banco. A prontidão retorna `503` quando o banco não responde. |
| Métricas operacionais | `GET /health/metrics` | Cabeçalho `x-ops-token` | Retorna métricas do processo e conexões do banco. Sem token configurado a rota responde `404`; token ausente ou inválido responde `403`. |
| Conta e sessão | `POST /api/auth/register`, `GET /api/auth/activate/:token`, `POST /api/auth/resend-activation`, `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me` | Cadastro, ativação, reenvio e login são públicos; logout e `/me` exigem sessão | Recebem dados de conta, token ou credenciais conforme a operação. Login cria cookie de sessão `HttpOnly`; `/me` retorna a conta e o perfil ativo. Cadastro retorna `201`; as demais operações retornam normalmente `200`. |
| Recuperação de senha | `POST /api/auth/forgot-password`, `POST /api/auth/reset-password` | Nenhuma; limite de requisições aplicado | Recebem e-mail ou token e nova senha com confirmação. A solicitação responde de forma neutra; token inválido, expirado ou reutilizado é recusado. |
| Perfil | `GET /api/users/me`, `PATCH /api/users/me/adult-content`, `PATCH /api/users/me/active-profile`, `PATCH /api/users/me/username`, `PATCH /api/users/me/password`, `DELETE /api/users/me` | Sessão válida; a troca de senha também é limitada por taxa | Consulta/alteração da própria conta, preferência adulta, perfil ativo, nome de usuário, senha ou exclusão. Corpos PATCH/DELETE contêm somente os campos definidos para a operação. |
| Catálogo pessoal | `GET /api/users/me/collection`, `GET /api/users/me/wishlist`, `GET /api/users/me/collection/editions/:editionId`; `PUT`, `PATCH` ou `DELETE` em `/collection/volumes/:volumeId`, `PUT` em `/collection/editions/:editionId`, e `PUT`/`DELETE` em `/wishlist/volumes/:volumeId` | Sessão válida | Consulta coleção, lista de desejos e seleção de volumes; altera posse, leitura e desejos individualmente ou em lote. Os lotes de coleção, lista de desejos e leitura são atômicos por edição, conforme RN31. |
| Catálogo público | `GET /api/public/catalog-options`, `/calendar`, `/works`, `/works/:slug`, `/authors/:authorId/works`, `/editions`, `/editions/:editionId`, `/volumes/:volumeId`, `/works/:slug/editions/:editionNumber` e `/works/:slug/editions/:editionNumber/volumes/:volumeNumber` | Anônima ou sessão opcional | Listas aceitam filtros, ordenação e paginação conforme o recurso; detalhes recebem identificador, slug ou contexto obra/edição/volume. Listas retornam dados e, quando paginadas, `page`, `limit`, `total` e `totalPages`. Registros privados ou não encontrados não são expostos. |
| Administração de usuários e opções | `GET /api/admin/users`, `PATCH /api/admin/users/:id/role`; `GET /api/admin/options/:category`, `POST /api/admin/options`, `PATCH`/`DELETE /api/admin/options/:id`; `GET /api/admin/works/form-options` e `/editions/form-options` | Sessão com perfil ativo Administrador | Consulta usuários/opções e administra papéis e valores de domínio. Recebe filtros por query ou os campos da entidade no corpo JSON. Criações retornam `201`; alterações e consultas, normalmente `200`. |
| Administração do catálogo | CRUD em `/api/admin/works`, `/api/admin/editions/:id` e `/api/admin/volumes/:id`; rotas aninhadas para criar/listar edições de obra e volumes de edição; `PATCH` de visibilidade de obra/edição | Sessão com perfil ativo Administrador | Corpos JSON seguem os schemas da entidade. Identificadores inteiros aparecem em `:id`; buscas contextuais usam slug/número. Criações retornam `201`; conflitos de integridade, duplicidade ou publicação inválida retornam erro apropriado. |
| Mídia administrativa | `POST /api/admin/media/covers`, `DELETE /api/admin/media/covers/:assetId` | Sessão com perfil ativo Administrador | Importa uma capa a partir de URL HTTPS e remove ativos pendentes. A importação valida e processa a imagem; o identificador do ativo é usado nas associações de Obra/Volume. |

As consultas administrativas por endereço contextual usam `GET /api/admin/works/slug/:slug`, `GET /api/admin/works/slug/:workSlug/editions/:editionNumber` e `GET /api/admin/works/slug/:workSlug/editions/:editionNumber/volumes/:volumeNumber`. As rotas CRUD também incluem `GET /api/admin/works/:id`, `GET /api/admin/editions/:id` e `GET /api/admin/volumes/:id`, além dos respectivos `PATCH` e `DELETE` quando disponíveis.

Os corpos das requisições e respostas são JSON UTF-8. Exemplo de listagem pública:

```http
GET /api/public/works?page=1&limit=24&sortBy=title&order=asc
```

```json
{
  "works": [],
  "pagination": { "page": 1, "limit": 24, "total": 0, "totalPages": 0 }
}
```

Exemplo de login: `POST /api/auth/login` recebe `{"email":"leitor@example.com","password":"senha"}`. Em caso de sucesso retorna `200`, define o cookie de sessão e responde com `{"message":"Login realizado com sucesso!","user":{"id":"…","username":"…","role":"…","profiles":[],"active_profile":"…"}}`. O cookie deve ser enviado nas próximas chamadas autenticadas; o frontend habilita credenciais HTTP.

As respostas de erro incluem `error` e `code`; validações locais podem incluir `field`. Códigos comuns: `400` entrada inválida, `401` sessão ou credenciais inválidas, `403` perfil/permissão ou política de conteúdo, `404` registro/rota não encontrado, `409` conflito de estado ou integridade, `429` limite de requisições, `500` erro interno, `502` falha de provedor e `503` dependência ou configuração indisponível. Alguns handlers têm códigos e respostas específicos; os schemas e os testes de cada rota são a referência detalhada do contrato.

### Integrações externas

| Integração | Finalidade | Autenticação | Dados trocados | Falhas previstas |
| --- | --- | --- | --- | --- |
| PostgreSQL gerenciado no Neon | Persistir contas, sessões, catálogo, opções e dados pessoais | URL de conexão em `DATABASE_URL`; acesso feito somente pelo backend via Prisma | Consultas e alterações relacionais; o frontend não recebe credenciais nem acessa o banco diretamente | Indisponibilidade/conexão falha resulta em erro da API; `/health/ready` responde `503`. |
| Cloudflare R2 | Armazenar capas processadas e remover objetos descartados | Credenciais S3 compatíveis do R2 mantidas no backend (`R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET`) | Backend envia bytes da imagem processada, tipo de conteúdo, cache e chave do objeto; solicita exclusão por chave. PostgreSQL guarda chaves e metadados, não os binários | Configuração ausente resulta em `503`; falha de armazenamento/remoção resulta em `502`. |
| Resend | Entregar e-mails transacionais de ativação e recuperação de senha | API key em `RESEND_API_KEY`, enviada como Bearer pelo backend; remetente verificado em `RESEND_FROM` | Backend envia destinatário, remetente, assunto e conteúdo textual/HTML com link de ação. Requisição HTTPS `POST https://api.resend.com/emails`, timeout de 10 segundos | Configuração ausente resulta em `503`; falha ou timeout do provedor resulta em `502`. Não há fila persistente nem repetição automática. |
| Vercel | Hospedar a SPA e encaminhar chamadas da interface à API | Rewrite de `/api/*` configurado em `vercel.json`; a sessão continua por cookie HTTP | Encaminha `/api/*` para `https://comanga-api.onrender.com/api/*`; não transmite acesso direto ao banco ou ao R2 | Indisponibilidade ou configuração incorreta do rewrite impede as chamadas à API. |

## 13. Pré-requisitos

| Ferramenta/Serviço | Versão | Obrigatório? | Observação |
| --- | --- | --- | --- |
| Git | Versão compatível com os repositórios | Sim | Necessário para obter os códigos-fonte. |
| Node.js | 22.12.0 ou superior | Sim | Requisito declarado nos dois `package.json`; o npm acompanha a instalação do Node.js. |
| PostgreSQL | Serviço acessível e compatível com o schema Prisma | Sim | A API precisa de um banco de desenvolvimento configurado em `DATABASE_URL`; o projeto usa Neon. Não use o banco de produção para desenvolvimento. |
| Conta e credenciais do Resend | — | Apenas para testar envio de e-mails | A API usa a API HTTP do Resend para ativação e recuperação de conta. Configure uma chave e um remetente de domínio verificado. |
| Credenciais do Cloudflare R2 | — | Apenas para importar ou remover capas | Necessárias às operações administrativas de mídia; não são necessárias para iniciar a interface e a API. |
| Docker | — | Não | Não é exigido pelo fluxo de desenvolvimento documentado. |

## 14. Instalação

Clone os repositórios e instale suas dependências separadamente. Os comandos abaixo pressupõem que ambos ficarão em diretórios irmãos:

```bash
git clone https://github.com/IsaacLeite1309/comanga-api.git
git clone https://github.com/IsaacLeite1309/comanga-web.git

cd comanga-api
npm ci

cd ../comanga-web
npm ci
```

`npm ci` instala as versões registradas nos arquivos `package-lock.json`. A geração do Prisma Client é feita na etapa de configuração, depois das migrations.

A instalação a partir de diretórios novos, seguindo a configuração e a execução das seções 15 e 16, foi concluída para a versão 0.9 — homologação, conforme confirmação do responsável. Não há log específico dessa validação anexado às evidências.

## 15. Configuração

Crie `comanga-api/.env` a partir de `comanga-api/.env.example` e informe os valores do ambiente de desenvolvimento. Use um banco Neon separado para desenvolvimento; não copie credenciais para a documentação ou para o Git.

| Variável | Obrigatória? | Descrição |
| --- | --- | --- |
| `DATABASE_URL` | Sim | URL de conexão com o PostgreSQL de desenvolvimento. |
| `SESSION_COOKIE_NAME` | Não | Nome do cookie de sessão; o padrão é `comanga_session`. |
| `FRONTEND_URL` | Não | Origem usada nos links enviados por e-mail; por padrão, acompanha `CORS_ORIGIN` ou `http://localhost:8080`. |
| `CORS_ORIGIN` | Não | Lista de origens permitidas separadas por vírgula; o padrão local é `http://localhost:8080`. |
| `RESEND_API_KEY` e `RESEND_FROM` | Para envio real de e-mail | Chave da API e endereço remetente associado a um domínio verificado no Resend. |
| `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET`, `MEDIA_PUBLIC_BASE_URL` | Para operações de capas | Acesso ao bucket do Cloudflare R2 e base pública das imagens. Mantenha os segredos somente no ambiente local ou no gerenciador de segredos do deploy. |
| `DATABASE_URL_TEST` | Somente para testes de integração | URL de um banco exclusivo de testes, com nome `test`, prefixo `test_` ou sufixo `_test`; não pode ser igual a `DATABASE_URL` nem `DIRECT_URL`. |
| `PORT` | Não | Porta HTTP da API; o padrão local é `3000`. |

No frontend, não é necessário criar `.env` para o desenvolvimento local padrão: o cliente usa `/api` e o Vite encaminha essa rota para `http://localhost:3000`. Se a API estiver em outro endereço, configure `VITE_API_PROXY_TARGET` para o proxy do Vite ou `VITE_API_URL` para a URL-base da API conforme o ambiente.

Depois de configurar `DATABASE_URL`, aplique as migrations ao banco de desenvolvimento e gere o cliente Prisma:

```bash
cd comanga-api
npm run migrate:dev
npm run prisma:generate
```

`migrate:dev` cria/aplica migrations no banco apontado pela configuração local. Confira a URL antes de executar e nunca use esse comando contra o banco de produção.

## 16. Execução

Inicie cada aplicação em um terminal, mantendo ambas em execução:

**Terminal 1 — API:**

```bash
cd comanga-api
npm run dev
```

A API inicia, por padrão, em `http://localhost:3000`. Consulte `http://localhost:3000/health/live` para verificar se o processo HTTP está ativo e `http://localhost:3000/health/ready` para verificar a prontidão, incluindo a conexão com o banco.

**Terminal 2 — frontend:**

```bash
cd comanga-web
npm run dev
```

Abra o endereço informado pelo Vite, normalmente `http://localhost:8080`. Durante o desenvolvimento, as chamadas `/api` são encaminhadas à API local. Para encerrar cada processo, use `Ctrl+C` no terminal correspondente.

## 17. Testes

O responsável confirmou a execução das ações dos casos CT-001 a CT-028 em 21/11/2026, no Windows com Google Chrome ([EV-008](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md)). A confirmação não identifica o commit nem inclui registros individuais das ações. A navegação pública foi verificada parcialmente em 25/11/2026 ([EV-001](../06-Evidencias/2026-11-25-v1/validacao-publica.md)); ela não cobre todos os passos de CT-001 a CT-028. Os resultados automatizados estão na seção 18.

### CT-001 — Cadastrar conta válida e rejeitar dados inválidos

- **Requisitos relacionados:** RF01; RN01, RN02, RN03.
- **Pré-condição:** Ambiente de teste com caixa de e-mail controlada e endereço ainda não cadastrado.
- **Dados de teste:** Nome de usuário exclusivo; e-mail controlado; nascimento válido; senha dentro da regra; variantes com nome/e-mail repetidos e senha inválida.
- **Passos:** 1. Cadastrar a conta válida. 2. Consultar seu estado e perfil atribuído. 3. Repetir o cadastro com cada variante inválida.
- **Resultado esperado:** A conta válida fica pendente, recebe o perfil padrão e inicia o envio da ativação; variantes inválidas não criam outra conta.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa. Em 25/11, observou-se adicionalmente: O formulário vazio exigiu os campos obrigatórios e o aceite dos termos; não houve criação de conta.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado. Observação pública adicional em 25/11/2026.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md); [EV-001 — observação pública parcial](../06-Evidencias/2026-11-25-v1/validacao-publica.md).

### CT-002 — Ativar conta e reenviar link

- **Requisitos relacionados:** RF02, RF03; RN03.
- **Pré-condição:** Conta pendente com acesso à caixa de e-mail de teste.
- **Dados de teste:** Link inicial, link reenviado e um token já usado ou expirado.
- **Passos:** 1. Solicitar reenvio. 2. Tentar o link inicial e depois o mais recente. 3. Repetir o uso do link válido e pedir novo reenvio após a ativação.
- **Resultado esperado:** O reenvio invalida o link anterior; o link recente ativa a conta uma vez; reutilização e reenvio para conta ativada são recusados.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-003 — Iniciar e encerrar sessão

- **Requisitos relacionados:** RF04, RF05; RN04; RNF03.
- **Pré-condição:** Conta ativa com senha conhecida e rota protegida acessível no ambiente de teste.
- **Dados de teste:** Credenciais válidas e senha incorreta.
- **Passos:** 1. Tentar acesso com senha incorreta. 2. Entrar com credenciais válidas e consultar a rota protegida. 3. Encerrar a sessão e repetir a consulta com o cookie anterior.
- **Resultado esperado:** A tentativa incorreta não cria sessão; a válida cria cookie `HttpOnly`; após o encerramento, a sessão não autoriza a rota.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-004 — Recuperar e alterar senha

- **Requisitos relacionados:** RF06; RN02, RN04, RN05.
- **Pré-condição:** Conta ativa com e-mail controlado; segunda sessão aberta para verificar revogação.
- **Dados de teste:** E-mail existente e inexistente; token recente; senha atual, senha nova válida e confirmação divergente.
- **Passos:** 1. Solicitar recuperação para os dois e-mails. 2. Redefinir a senha com token válido e tentar reutilizá-lo. 3. Entrar e alterar a senha no perfil, primeiro com senha atual incorreta e depois correta.
- **Resultado esperado:** A solicitação não revela a existência da conta; token usado não funciona novamente; confirmação ou senha atual incorreta impede alteração; a senha antiga e as sessões aplicáveis deixam de valer.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-005 — Consultar perfil, alterar nome e trocar perfil ativo

- **Requisitos relacionados:** RF07, RF08; RN01, RN06; RNF04.
- **Pré-condição:** Conta com perfis padrão e administrador atribuídos, aberta em duas abas.
- **Dados de teste:** Novo nome de usuário exclusivo e nome já utilizado por outra conta.
- **Passos:** 1. Consultar o perfil e alterar o nome. 2. Tentar o nome duplicado. 3. Alternar o perfil ativo e observar menus e acesso nas duas abas.
- **Resultado esperado:** A consulta e alteração afetam apenas a própria conta; nome duplicado é recusado; menus e permissões acompanham o perfil ativo em ambas as abas.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-006 — Controlar preferência de conteúdo adulto

- **Requisitos relacionados:** RF09; RN08, RN17.
- **Pré-condição:** Contas de teste maior e menor de 18 anos; obra pública adulta e obra pública não adulta.
- **Dados de teste:** Datas de nascimento imediatamente antes e depois do 18º aniversário.
- **Passos:** 1. Tentar ativar a preferência na conta menor. 2. Ativá-la na conta maior. 3. Comparar a busca e o acesso direto à obra adulta como visitante e nas duas contas.
- **Resultado esperado:** Menor e visitante não recebem conteúdo adulto; maior com preferência ativa pode recebê-lo; o limite de idade respeita o aniversário completo.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-007 — Excluir conta e dados pessoais

- **Requisitos relacionados:** RF10; RN04, RN06, RN20.
- **Pré-condição:** Conta descartável com sessão ativa, volume na coleção e volume na lista de desejos.
- **Dados de teste:** Senha correta e incorreta da conta descartável.
- **Passos:** 1. Tentar exclusão com senha incorreta. 2. Excluir com a senha correta. 3. Consultar a sessão anterior e os vínculos pessoais pelo banco de teste.
- **Resultado esperado:** Senha incorreta não exclui; a exclusão remove conta, sessões e vínculos pessoais sem afetar outras contas.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-008 — Consultar usuários e administrar perfis

- **Requisitos relacionados:** RF11; RN06, RN07; RNF04.
- **Pré-condição:** Duas contas administradoras e uma conta padrão no ambiente de teste; nenhuma alteração deve deixar o sistema sem administrador efetivo.
- **Dados de teste:** Identificadores das contas e filtros de consulta.
- **Passos:** 1. Consultar e filtrar usuários. 2. Atribuir e retirar o perfil administrador de outra conta. 3. Tentar alterar a própria atribuição e conferir que as mudanças autorizadas mantiveram ao menos um administrador efetivo.
- **Resultado esperado:** A listagem mostra perfis atribuídos; alterações autorizadas funcionam; automodificação é recusada e o estado final preserva ao menos um administrador efetivo.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-009 — Manter opções administrativas

- **Requisitos relacionados:** RF12; RN09, RN16.
- **Pré-condição:** Administrador ativo e categoria editável com um valor de teste não utilizado.
- **Dados de teste:** Rótulo novo e variante com diferença apenas de caixa.
- **Passos:** 1. Criar, consultar e alterar o valor. 2. Tentar cadastrar a variante duplicada. 3. Vincular o valor a uma obra ou edição e tentar excluí-lo.
- **Resultado esperado:** Valor válido aparece nos formulários; duplicidade é recusada; valor referenciado não é excluído.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-010 — Importar e substituir capa interna

- **Requisitos relacionados:** RF13; RN23, RN24; RNF10.
- **Pré-condição:** Administrador ativo, obra ou volume de teste com capa associada e imagem HTTPS válida disponível.
- **Dados de teste:** URL HTTPS de imagem válida; URL local ou privada; conteúdo inválido; imagem acima do limite configurado.
- **Passos:** 1. Importar a imagem válida e associá-la. 2. Tentar as origens e conteúdos inválidos. 3. Substituir a capa existente e consultar a URL pública derivada.
- **Resultado esperado:** Somente mídia válida é armazenada e servida pela infraestrutura interna; falha preserva a capa anterior; substituição concluída mostra a nova capa.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-011 — Manter obra e validar seus dados mínimos

- **Requisitos relacionados:** RF14; RN09, RN10, RN25.
- **Pré-condição:** Administrador ativo; tipo, autor, capa e opções compatíveis cadastrados para teste.
- **Dados de teste:** Obra com campos obrigatórios, sem título original; variante sem autor; variante com título nacional duplicado.
- **Passos:** 1. Criar e consultar a obra válida. 2. Alterar apenas sua sinopse e verificar os demais campos. 3. Tentar criar as variantes inválidas e excluir a obra enquanto privada e sem edições.
- **Resultado esperado:** A obra válida inicia privada e mantém campos não alterados; ausência de autor ou título duplicado é recusada; a obra elegível pode ser excluída.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-012 — Aplicar regras editoriais e classificação da obra

- **Requisitos relacionados:** RF14; RN09, RN26, RN27.
- **Pré-condição:** Administrador ativo; opções de revista, demografia e gêneros com códigos `boys-love`, `girls-love` e `hentai` disponíveis no ambiente de teste.
- **Dados de teste:** Obra com início/fim original; revista, demografia e gênero; dois papéis conflitantes para o mesmo autor.
- **Passos:** 1. Informar fim anterior ao início e papéis conflitantes. 2. Marcar lançamento direto em obra com revista e demografia. 3. Associar cada gênero especial e consultar a classificação persistida.
- **Resultado esperado:** Período e papéis inválidos são recusados; lançamento direto limpa revista e demografia; gêneros sem demografia limpam esse vínculo; `hentai` força classificação adulta.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-013 — Manter edição e obter capa derivada

- **Requisitos relacionados:** RF15; RN11, RN28, RN30.
- **Pré-condição:** Obra de teste existente; editora brasileira e opções físicas válidas.
- **Dados de teste:** Edição com número positivo, editora e status; acabamento/formato ausentes; miolos vazios ou duplicados; volume 1 com capa.
- **Passos:** 1. Criar, consultar e alterar a edição privada. 2. Tentar número repetido, editora ausente e miolo duplicado. 3. Cadastrar volume 1 e consultar a capa e o total; criar outra edição privada sem volumes e excluí-la.
- **Resultado esperado:** A edição válida inicia privada; entradas inválidas são recusadas; sua capa vem do volume 1 e seu total reflete os volumes vinculados; edição elegível pode ser excluída.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-014 — Manter volume, data e ISBN

- **Requisitos relacionados:** RF16; RN12, RN13, RN29.
- **Pré-condição:** Duas edições privadas de teste, uma para volumes numerados e outra vazia para testar volume único; capa interna disponível.
- **Dados de teste:** Volume 0; volume 1; volume único; datas com precisões completa, mês/ano e ano; ISBN válido e inválido.
- **Passos:** 1. Criar, consultar e alterar volumes numerados na primeira edição. 2. Tentar número repetido, data impossível e ISBN incorreto nela. 3. Cadastrar volume único na segunda edição e tentar adicionar outro; excluir um volume privado elegível na primeira.
- **Resultado esperado:** Número 0 e datas válidas são aceitos; dados inválidos e coexistência com volume único são recusados; volume único recebe número 1.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-015 — Controlar visibilidade e proteger exclusões

- **Requisitos relacionados:** RF17; RN11, RN14, RN15.
- **Pré-condição:** Obra privada de teste com edição privada e volume 1 com capa; conta descartável para coleção e lista de desejos.
- **Dados de teste:** Identificadores da obra, edição e volume; visibilidades Público e Privado; vínculos de posse e desejo.
- **Passos:** 1. Tentar publicar a edição antes da obra e depois publicar a obra. 2. Conferir a visibilidade dos volumes e tentar excluir registros públicos. 3. Adicionar um volume à coleção e tentar tornar a edição e a obra privadas. 4. Repetir com vínculo de desejo. 5. Remover os vínculos e tornar a edição privada.
- **Resultado esperado:** A edição só é publicada com obra pública e volume 1 apto; seus volumes acompanham a mudança. Exclusões públicas são recusadas. Edição ou obra não podem ser tornadas privadas enquanto houver vínculo pessoal protegido; sem vínculos, a alteração elegível é concluída.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa. A verificação manual de 21/11 não comprova os passos da proteção de visibilidade implementada em 25/11; a regra atual foi coberta por testes automatizados.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL; passos posteriores cobertos por testes automatizados.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md); [EV-007 — testes automatizados](../06-Evidencias/2026-11-25-v1/api-integracao-ci.log).

### CT-016 — Explorar, pesquisar e filtrar obras e edições

- **Requisitos relacionados:** RF18; RN17, RN18; RNF01.
- **Pré-condição:** Catálogo com obras e edições públicas, privadas e classificações diferentes.
- **Dados de teste:** Termo compartilhado por alguns registros; dois filtros compatíveis; limite de 51 itens para a API paginada.
- **Passos:** 1. Abrir listagens de obras e edições sem termo. 2. Aplicar termo, filtros combinados e ordenação; trocar entre as duas listagens. 3. Consultar URL privada e requisitar `limit=51`.
- **Resultado esperado:** Listagens funcionam sem termo; filtros produzem a interseção e preservam o termo compatível; registros privados não aparecem; limite acima de 50 é recusado.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa. Em 25/11, observou-se adicionalmente: A pesquisa por One Piece retornou 8 obras; combinada com o tipo Databook, retornou 5.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado. Observação pública adicional em 25/11/2026.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md); [EV-001 — observação pública parcial](../06-Evidencias/2026-11-25-v1/validacao-publica.md).

### CT-017 — Navegar por obra, edição, volume e autor

- **Requisitos relacionados:** RF19, RF20; RN17, RN30.
- **Pré-condição:** Obra pública com autor, edição pública e dois volumes públicos.
- **Dados de teste:** Endereços públicos da obra, edição, volume e autor; um endereço privado.
- **Passos:** 1. Navegar da obra até edição, volume e autor. 2. Usar os links contextuais de retorno. 3. Comparar o total exibido com os volumes públicos e acessar o endereço privado.
- **Resultado esperado:** As páginas mostram somente dados públicos e relações corretas; total é calculado dos volumes públicos; o endereço privado não revela dados.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa. Em 25/11, observou-se adicionalmente: Foram abertas as fichas públicas de One Piece, sua 3ª edição, volume 1 e a página de Eiichiro Oda.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado. Observação pública adicional em 25/11/2026.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md); [EV-001 — observação pública parcial](../06-Evidencias/2026-11-25-v1/validacao-publica.md).

### CT-018 — Consultar calendário mensal de lançamentos

- **Requisitos relacionados:** RF21; RN13, RN19.
- **Pré-condição:** Volumes públicos de teste com data completa, mês/ano e somente ano; volume privado no mesmo período.
- **Dados de teste:** Mês e ano específicos; termo e editora brasileira.
- **Passos:** 1. Abrir lançamentos sem escolher período. 2. Selecionar outro mês, aplicar termo e editora. 3. Verificar volumes com data apenas anual ou visibilidade privada.
- **Resultado esperado:** A abertura usa o mês corrente; filtros restringem resultados; datas apenas anuais não entram em mês arbitrário e volumes privados não aparecem.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa. Em 25/11, observou-se adicionalmente: A tela de lançamentos abriu com 22 volumes; a busca por Chainsaw retornou o volume 24 e a troca de mês preservou o termo.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado. Observação pública adicional em 25/11/2026.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md); [EV-001 — observação pública parcial](../06-Evidencias/2026-11-25-v1/validacao-publica.md).

### CT-019 — Adicionar e remover volume da coleção

- **Requisitos relacionados:** RF22; RN20, RN21.
- **Pré-condição:** Conta padrão ativa; volume público inicialmente na lista de desejos e fora da coleção.
- **Dados de teste:** Identificador do volume e estados lido/não lido.
- **Passos:** 1. Adicionar o volume à coleção. 2. Marcá-lo como lido e consultar as telas da obra, edição, volume e coleção. 3. Removê-lo da coleção e consultar novamente.
- **Resultado esperado:** Inclusão remove o desejo e inicia como não lido; leitura aparece nas telas; remoção apaga o vínculo e seu estado de leitura.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa. Em 25/11, observou-se adicionalmente: A tentativa de adicionar o volume 1 de One Piece sem autenticação exibiu pedido de login e não marcou o volume.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado. Observação pública adicional em 25/11/2026.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md); [EV-001 — observação pública parcial](../06-Evidencias/2026-11-25-v1/validacao-publica.md).

### CT-020 — Alterar coleção em lote de forma atômica

- **Requisitos relacionados:** RF23; RN20, RN21, RN22, RN31; RNF06.
- **Pré-condição:** Conta ativa e edição pública com pelo menos três volumes públicos.
- **Dados de teste:** Dois IDs válidos da edição e um ID inválido ou de outra edição.
- **Passos:** 1. Incluir e remover volumes válidos na seleção e salvar. 2. Repetir a operação com um ID inválido no mesmo lote. 3. Consultar os vínculos da coleção após cada tentativa.
- **Resultado esperado:** Lote válido grava a seleção; lote com qualquer volume inválido não altera nenhum vínculo anterior.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-021 — Manter lista de desejos individual

- **Requisitos relacionados:** RF24; RN20, RN21.
- **Pré-condição:** Conta ativa; um volume público fora da coleção e outro já incluído nela.
- **Dados de teste:** Identificadores dos dois volumes.
- **Passos:** 1. Adicionar o primeiro à lista de desejos e consultar a lista. 2. Tentar incluir o volume que está na coleção. 3. Remover o primeiro da lista.
- **Resultado esperado:** O primeiro volume entra e sai da lista; o volume da coleção é recusado; a lista reflete cada alteração válida.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-022 — Consultar coleção e lista de desejos com privacidade

- **Requisitos relacionados:** RF25; RN20, RN21.
- **Pré-condição:** Duas contas com acervos distintos; uma delas possui vínculo com volume de edição pública.
- **Dados de teste:** Termo, filtro e ordenação disponíveis nas duas telas.
- **Passos:** 1. Consultar coleção e lista em ambas as contas. 2. Aplicar busca, filtros e ordenação. 3. Tentar tornar privada a edição do volume vinculado e repetir as consultas.
- **Resultado esperado:** Cada conta vê somente seus vínculos e totais; filtros alteram o resultado; a mudança para privada é recusada enquanto houver vínculo pessoal protegido.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa. A verificação manual de 21/11 não comprova os passos da proteção de visibilidade implementada em 25/11; a regra atual foi coberta por testes automatizados.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL; passos posteriores cobertos por testes automatizados.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md); [EV-007 — testes automatizados](../06-Evidencias/2026-11-25-v1/api-integracao-ci.log).

### CT-023 — Alterar lista de desejos em lote de forma atômica

- **Requisitos relacionados:** RF26; RN20, RN21, RN31.
- **Pré-condição:** Conta ativa e edição pública com pelo menos três volumes, um deles já presente na coleção.
- **Dados de teste:** Duas alterações válidas na lista de desejos e uma tentativa inválida envolvendo o volume da coleção.
- **Passos:** 1. Salvar a seleção válida em lote. 2. Submeter uma alteração em lote que inclua, por chamada à API de teste, o volume já pertencente à coleção. 3. Comparar todos os vínculos antes e depois da tentativa inválida.
- **Resultado esperado:** Seleção válida é aplicada integralmente; lote inválido não altera nenhum vínculo da lista.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-024 — Alterar estado de leitura em lote de forma atômica

- **Requisitos relacionados:** RF27; RN20, RN21, RN31.
- **Pré-condição:** Conta ativa e edição pública com dois volumes na coleção e um fora dela.
- **Dados de teste:** Alterações de leitura válidas e tentativa que inclua o volume fora da coleção.
- **Passos:** 1. Marcar os volumes da coleção como lidos e salvar. 2. Submeter, pela API de teste, um lote com alteração do volume fora da coleção. 3. Comparar todos os estados de leitura antes e depois da tentativa inválida.
- **Resultado esperado:** Lote válido altera todos os estados; lote inválido não altera nenhum estado, e volume fora da coleção continua sem leitura.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-025 — Conferir cache e atualização das telas

- **Requisitos relacionados:** RF18, RF19, RF22, RF25; RNF08.
- **Pré-condição:** Navegador com sessão ativa e volumes públicos consultáveis em pesquisa, edição, coleção e lista de desejos.
- **Dados de teste:** Uma busca com filtros definidos e um volume cuja posse será alterada.
- **Passos:** 1. Abrir e revisitar as páginas sem mudar os filtros. 2. Alterar a posse do volume. 3. Voltar às telas relacionadas e mudar um filtro.
- **Resultado esperado:** Consultas reutilizam entradas dentro do período configurado; posse e contadores afetados se atualizam; novo filtro produz resultado próprio.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-026 — Verificar segurança de credenciais, sessão e login

- **Requisitos relacionados:** RF01, RF04; RN02, RN04; RNF02, RNF03, RNF05.
- **Pré-condição:** Conta ativa descartável e acesso autorizado ao banco de teste e aos cabeçalhos HTTP.
- **Dados de teste:** Senha válida; cinco tentativas de login incorretas no mesmo IP.
- **Passos:** 1. Conferir que a senha persistida é hash bcrypt e que o token de sessão é guardado como hash. 2. Examinar os atributos do cookie de sessão. 3. Repetir falhas de login até o bloqueio.
- **Resultado esperado:** Não há senha ou token de sessão em texto puro no banco; cookie usa `HttpOnly` e configuração `Secure` conforme o ambiente; o limitador responde HTTP 429 após o limite.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-027 — Verificar integridade e tratamento de erro

- **Requisitos relacionados:** RF14, RF17, RF23; RN14, RN22; RNF06, RNF07.
- **Pré-condição:** Banco de teste com obra, edição, volumes e conta descartáveis; acesso aos logs da API.
- **Dados de teste:** Operação composta com falha de validação e requisição que provoque erro de servidor controlado.
- **Passos:** 1. Submeter a operação composta inválida. 2. Consultar os registros relacionados. 3. Provocar o erro controlado e examinar resposta e log pelo identificador da requisição.
- **Resultado esperado:** A operação inválida não deixa dados parciais; erro da API retorna JSON sem stack trace; log estruturado registra o contexto da falha.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

### CT-028 — Verificar navegação, capas e estados vazios

- **Requisitos relacionados:** RF18, RF19, RF21, RF25; RNF09, RNF10, RNF11.
- **Pré-condição:** Interface acessível em navegador com ferramentas de largura de tela; obra com capa e filtros que não retornem registros.
- **Dados de teste:** Larguras de 767 e 768 px; uma capa indisponível; filtro sem resultados.
- **Passos:** 1. Abrir pesquisa, lançamentos, coleção e lista nas duas larguras. 2. Observar carregamento e resultado vazio. 3. Induzir falha de carregamento da capa.
- **Resultado esperado:** A navegação principal usa barra inferior abaixo de 768 px e barra lateral a partir de 768 px; as páginas mostram carregamento e estado vazio; capa mantém proporção 2:3 e exibe placeholder na falha.
- **Resultado obtido:** O responsável confirmou que realizou as ações em 21/11/2026; não foram fornecidos resultados detalhados por etapa.
- **Status:** AÇÕES CONFIRMADAS PELO RESPONSÁVEL.
- **Ambiente e versão:** Windows e Google Chrome; 21/11/2026. Commit não informado.
- **Evidência:** [EV-008 — confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md).

## 18. Evidências

As evidências identificam o ambiente, a data e o escopo observado em cada validação.

| ID | Tipo | Ambiente e data | Resultado | Evidência |
| --- | --- | --- | --- | --- |
| EV-001 | Validação manual parcial | Site público, sessão anônima; 25/11/2026 | Pesquisa, navegação por obra/edição/volume, lançamentos e bloqueio de adição anônima observados. Não cobre todos os casos manuais. | [Registro das ações e resultados](../06-Evidencias/2026-11-25-v1/validacao-publica.md) |
| EV-002 | Testes unitários da API | Windows; commit 23b59c4; 25/11/2026 | **57 suítes e 792 testes aprovados**; cobertura: instruções 92,01%, ramificações 83,70%, funções 91,15% e linhas 93,52%. Lint, arquitetura e build concluídos. | [Testes e cobertura](../06-Evidencias/2026-11-25-v1/api-unitarios.log); [verificações](../06-Evidencias/2026-11-25-v1/comanga-api-verificacoes.log) |
| EV-003 | Testes do frontend | Windows; commit 81c3e38; 25/11/2026 | **69 arquivos e 628 testes aprovados**; cobertura: instruções 91,21%, ramificações 86,59%, funções 91,41% e linhas 93,53%. Lint, arquitetura e build concluídos. Workflow do GitHub Actions aprovado. | [Testes e cobertura](../06-Evidencias/2026-11-25-v1/frontend-testes.log); [verificações](../06-Evidencias/2026-11-25-v1/comanga-web-verificacoes.log); [log do CI](../06-Evidencias/2026-11-25-v1/frontend-ci.log); [GitHub Actions](https://github.com/IsaacLeite1309/comanga-web/actions/runs/45690820696) |
| EV-004 | Suíte completa e reteste da API no Neon | Windows; banco de teste `comanga_test`; commit `23b59c4`; 25/11/2026 | Na execução completa, **94 de 95 suítes e 1.299 de 1.300 testes aprovados**; um teste de migração excedeu o limite de 30 segundos. No reteste isolado, **4 de 4 testes passaram em 45,5 s**, com limite de 180 segundos. A execução completa terminou com falha; o reteste não equivale a uma nova execução integral aprovada. | [Log da suíte completa](../06-Evidencias/2026-11-25-v1/api-integracao-neon.log); [log do reteste](../06-Evidencias/2026-11-25-v1/api-neon-reteste-migracao.log); [configuração do reteste](../06-Evidencias/2026-11-25-v1/jest-neon-reteste.js.txt) |
| EV-005 | Restauração do backup PostgreSQL | CoMangá 0.9 — homologação; 03/11/2026; Windows; `pg_restore` 18.6; branch temporária do Neon; banco `neondb` | **Restauração concluída com sucesso.** O SHA-256 do dump correspondeu ao arquivo de verificação. Após a restauração, o Neon SQL Editor listou as 22 tabelas esperadas e confirmou dados acessíveis: `works` 177, `editions` 186, `volumes` 595, `users` 10, `domain_option_values` 1.529 e `_prisma_migrations` 47. | Evidência registrada nesta linha: compatibilidade SHA-256 e resultados das consultas pós-restauração. O procedimento terminou com o banco restaurado e consultável. |
| EV-006 | Restauração das capas do Cloudflare R2 | CoMangá 0.9 — homologação; 03/11/2026; Windows PowerShell; `rclone` 1.75.2; bucket de teste `comanga-capas-restore-teste` | **Restauração concluída com sucesso.** `rclone copy` transferiu os 2.517 objetos de `comanga-backups/media/current` (319,229 MiB). A verificação `rclone check --one-way --size-only` terminou com **0 diferenças** e **2.517 arquivos correspondentes**. Um objeto de capa foi baixado do bucket restaurado e aberto com sucesso. | Evidência registrada nesta linha: resultados de cópia, verificação integral dos objetos e abertura de uma capa. |
| EV-007 | Suíte completa da API no GitHub Actions | Linux; PostgreSQL 16 isolado; commit `23b59c4`; 25/11/2026 | **95 suítes e 1.300 testes aprovados**; cobertura: instruções 95,22%, ramificações 88,49%, funções 94,69% e linhas 96,16%. Não comprova o resultado da suíte no Neon. | [Log completo](../06-Evidencias/2026-11-25-v1/api-integracao-ci.log); [execução no GitHub Actions](https://github.com/IsaacLeite1309/comanga-api/actions/runs/45690820953) |
| EV-008 | Confirmação dos testes manuais | Windows; Google Chrome; 21/11/2026; commit não informado | O responsável confirmou a execução das ações de CT-001 a CT-028. Passos relativos à proteção de visibilidade adicionada após essa data são sustentados pelos testes automatizados, sem alegação de execução manual em 21/11. | [Confirmação do responsável](../06-Evidencias/2026-11-25-v1/confirmacao-testes-manuais.md) |

O [índice das evidências](../06-Evidencias/2026-11-25-v1/README.md) identifica os arquivos de teste por área e inclui um manifesto com os hashes dos registros concluídos.

## 19. Deploy

O código é publicado a partir dos repositórios da API e do frontend. A API deve estar compatível com o esquema do banco antes da publicação do frontend que dependa de novos contratos. Os comandos abaixo pertencem ao respectivo repositório; credenciais de produção ficam nas configurações das plataformas.

| Etapa | Comando/Procedimento | Responsável | Validação |
| --- | --- | --- | --- |
| Preparação e build | Após a revisão e aprovação do código, executar `npm ci`, `npm run prisma:generate` e `npm run check` na API; no frontend, `npm ci` e `npm run check`. O comando `check` inclui build e a cobertura mínima configurada. | Isaac e Diogo | Conferir a conclusão dos workflows de qualidade dos dois repositórios e a geração de `dist` na API e no frontend. |
| Backup antes de alteração de dados | Antes de migrations ou alterações em massa, criar snapshot do banco de produção no Neon e conferir o backup externo, quando o workflow de backup estiver configurado. | Isaac | Registrar o ponto de recuperação e conferir a existência do snapshot ou do arquivo de backup; o procedimento está em `comanga-api/docs/operations/production-backups.md`. |
| Migrations | No ambiente autorizado da API, executar `npm run check:covers` quando a mudança afetar capas e aplicar as migrations versionadas com `npm run migrate` antes de liberar uma versão que dependa delas. Nunca usar `prisma migrate reset` em produção nem editar migrations já aplicadas. | Isaac | Confirmar que `prisma migrate deploy` concluiu sem erro e que `GET /health/ready` responde com sucesso. |
| Publicação da API | Promover a revisão aprovada para `main` e publicar a API na Render com Prisma Client gerado, TypeScript compilado e inicialização por `npm start`. Manter `DATABASE_URL`, Resend e R2 nas variáveis da Render. | Isaac | Conferir logs de inicialização, `GET /health/live`, `GET /health/ready` e uma operação de leitura do catálogo. |
| Publicação do frontend | Promover a revisão aprovada para `main` e publicar o build Vite na Vercel. Preservar os rewrites de `vercel.json`, inclusive `/api/:path*` para a API na Render. | Isaac | Abrir o site publicado, carregar pesquisa e detalhes, autenticar uma conta de teste e conferir uma chamada `/api` no navegador. |
| Validação após publicação | Comparar versão publicada, migrations aplicadas e commits promovidos; conferir catálogo público, login, operações administrativas e carregamento de capas. | Isaac e Diogo | Registrar data, ambiente, commit e resultado do roteiro de homologação; investigar qualquer falha antes de encerrar a publicação. |
| Rollback | Se a nova versão falhar, reimplantar na Render e na Vercel os últimos commits funcionais compatíveis com o esquema atual. Migrations aplicadas não são revertidas automaticamente: se houver incompatibilidade de dados ou esquema, preparar uma migration corretiva ou restaurar um backup validado com plano de preservação de dados. | Isaac | Repetir verificações de saúde e fluxos essenciais; confirmar que a versão anterior não depende de colunas ou contratos removidos. |

## 20. Segurança

| Controle | Tratamento adotado |
| --- | --- |
| Senhas e tokens | Senhas são armazenadas como hash bcrypt, com fator de trabalho 10. O identificador de sessão é entregue por cookie `HttpOnly` e somente seu hash é persistido no banco; tokens de recuperação têm prazo e uso único. |
| Sessão e autorização | O cookie recebe `Secure` em produção. Rotas administrativas exigem sessão válida, atribuição administrativa vigente e perfil de administrador ativo. O backend aplica também as regras de visibilidade e acesso a conteúdo adulto. |
| Dados pessoais | Consultas da coleção, da lista de desejos e do perfil usam a identidade da sessão, sem aceitar que o cliente escolha o titular. Respostas públicas omitem registros privados e conteúdo adulto sem autorização. |
| Proteção contra tentativas repetidas | O login limita falhas por IP a cinco tentativas em cinco minutos, respondendo `429` durante a janela. O contador fica na memória do processo da API. |
| Segredos e configuração | `.env.example` contém nomes de variáveis e valores de exemplo sem credenciais reais. `DATABASE_URL`, chaves do R2 e `RESEND_API_KEY` devem ficar somente no ambiente local autorizado e nos gerenciadores de variáveis das plataformas. |
| Erros e logs | Respostas de erro não incluem stack trace. A API registra evento, rota, status, duração e identificador da requisição; em produção, o logger omite o stack trace. Logs e evidências devem ser revisados antes da publicação para evitar dados sensíveis. |
| Mídia externa | A importação de capas é administrativa, usa URL HTTPS validada e armazena a imagem processada no Cloudflare R2. O banco guarda somente chaves e metadados do arquivo. |

## 21. Limitações conhecidas

| ID | Limitação | Impacto | Alternativa/Plano futuro |
| --- | --- | --- | --- |
| LIM-01 | O envio de e-mails pelo Resend não usa fila persistente nem repetição automática. | Falhas temporárias do provedor podem impedir a entrega de ativação ou recuperação nessa tentativa. | Conferir a resposta e os logs da API, corrigir a configuração e solicitar novo envio; avaliar fila e tentativas automáticas numa evolução. |
|  |
|  |
| LIM-02 | A resposta do sistema depende de Render, Neon, Vercel, Cloudflare R2 e Resend. | Indisponibilidade ou latência de um desses serviços pode atrasar páginas, impedir gravações, ocultar capas ou interromper mensagens de conta. | Usar as rotas de saúde, logs e painéis dos provedores para identificar a dependência afetada; tratar recuperação e retentativas conforme o serviço. |
|  |
|  |

## 22. Troubleshooting

| Sintoma | Causa provável | Verificação | Solução segura |
| --- | --- | --- | --- |
| API não inicia ou `/health/ready` responde `503` | Banco Neon indisponível, URL de conexão incorreta ou falha de rede. | Conferir `/health/live`, logs da Render, estado do Neon e variáveis de conexão no ambiente afetado, sem expor a URL completa. | Corrigir a configuração ou aguardar a recuperação da dependência; reiniciar a API somente depois de restabelecer a conexão. |
| Login ou página autenticada responde `401` | Credenciais incorretas, sessão encerrada ou cookie ausente. | Conferir a resposta de `/api/auth/me`, os atributos do cookie e se `/health/ready` está saudável. | Entrar novamente com credenciais válidas; se o banco estiver indisponível, tratar a falha da dependência antes de invalidar sessões. |
| Área administrativa responde `403` | A conta não tem atribuição administrativa ou está com o perfil padrão ativo. | Consultar o perfil ativo e os perfis atribuídos à conta. | Se a conta possui a atribuição, selecionar Administrador na página de perfil; caso contrário, solicitar a concessão a outro administrador. |
| Tentativas de login recebem `429` | Limite de falhas por IP atingido. | Conferir o horário das falhas e a janela de cinco minutos nos logs. | Aguardar o fim da janela e tentar com credenciais corretas; investigar repetição anormal de tentativas. |
| Capa ausente ou importação falha | URL de origem inválida, falha de processamento, configuração do R2 ou objeto indisponível. | Conferir o erro da requisição, os logs da API, a configuração do R2 e executar a checagem sem escrita `npm run check:covers` quando pertinente. | Corrigir a origem ou a configuração e repetir a importação pelo painel; não remover objetos do bucket manualmente sem verificar referências. |
| E-mail de ativação ou recuperação não chega | Chave, remetente ou domínio do Resend incorreto, ou falha do provedor. | Conferir status retornado pela API, logs de envio, variáveis `RESEND_API_KEY` e `RESEND_FROM` e estado do domínio verificado. | Corrigir a configuração e solicitar novo envio; não publicar a chave em tickets ou logs. |
| Frontend carrega, mas chamadas `/api` falham | Rewrite da Vercel incorreto, API na Render indisponível ou contrato diferente da versão publicada. | Conferir `vercel.json`, rede do navegador, URL de destino e `/health/ready` da API. | Corrigir o rewrite ou promover versões compatíveis da API e do frontend; validar pesquisa e autenticação após a publicação. |
| Migration falha | Dados legados incompatíveis, conexão com banco errado ou histórico divergente. | Conferir o erro do `prisma migrate deploy`, o banco selecionado, o snapshot e o histórico das migrations. | Preservar os dados, corrigir a incompatibilidade por nova migration e repetir a aplicação; não usar `prisma migrate reset` em produção. |

## 23. Histórico de versões

| Versão | Data | Mudanças principais | Responsável |
| --- | --- | --- | --- |
| 0.9 — homologação | 21/10/2026 | Catálogo público e administrativo de obras, edições e volumes; autenticação e perfis; coleção, leitura, desejos e lançamentos; documentação técnica e evidências da N2. | Isaac Leite e equipe CoMangá. |
| 1.0 | 25/11/2026 | Identificação da versão atual do sistema e atualização da documentação; evidências anteriores mantidas com suas versões e datas originais. | Isaac Leite e equipe CoMangá. |

## 24. Responsáveis

| Pessoa | Papel | Responsabilidades | Contato institucional |
| --- | --- | --- | --- |
| Isaac Leite | Líder, desenvolvedor e testes | Planejamento, coordenação, integração das entregas, desenvolvimento da API e do frontend, validação e publicação. | 2210885@unicesusc.edu.br |
| Diogo Vieira | Desenvolvedor e testes | Desenvolvimento da API e do frontend, revisão e testes das funcionalidades. | 2510989@unicesusc.edu.br |
| Higor Pessoa | Curador do catálogo | Pesquisa e cadastro de obras, edições e volumes de mangás no sistema. | 2420399@unicesusc.edu.br |
| Klaudenilson Alves | Curador do catálogo | Pesquisa e cadastro de obras, edições e volumes de mangás no sistema. | 2510957@unicesusc.edu.br |

## Registro de revisão

Esta edição documenta o sistema identificado como versão 1.0. As ações manuais de 21/11 foram confirmadas pelo responsável; o commit utilizado não foi informado. A proteção de visibilidade implementada posteriormente foi coberta por testes automatizados. A execução completa da API no Neon apresentou uma falha por tempo limite, seguida de reteste isolado aprovado.
