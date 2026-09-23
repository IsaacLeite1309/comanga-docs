# SERS - CoMangá

Grupo: Isaac Leite, Lucas Dalla, Klau Alves, Gabriel Mourgues, Higor Pessoa

## 1. Introdução

**Propósito:** este documento define os Requisitos Funcionais (RF), as Regras de Negócio (RN) e os Requisitos Não Funcionais (RNF) do CoMangá. Ele descreve o sistema implementado nas branches integradas da API (`comanga-api`) e da Web (`comanga-web`) e separa, de forma explícita, o que já existe, o que está planejado e o que foi cancelado. O objetivo do projeto é criar a plataforma de referência para catalogação e acompanhamento de mangás publicados no Brasil.

### 1.1. Estados dos requisitos

Cada RF, RN e RNF traz um campo **Estado**:

- **Vigente:** requisito aplicável à etapa atual do produto. Esse estado não comprova implementação completa nem validação aprovada.
- **Futuro (planejado):** aprovado para uma próxima etapa, mas sem implementação na API. Pode haver apenas telas “Em breve” na Web.
- **Cancelado:** abandonado. O identificador é mantido apenas para rastreabilidade e não pode ser reutilizado.

O campo Estado traz apenas um desses três valores, sem histórico de alterações. O grau de atendimento é informado separadamente: comportamento implementado e limitações na descrição, divergências na seção 6.3 e, para os RNFs, situação e evidências nos cenários ATAM. Metas sem medição ou sem procedimento comprovado, como RNF01, RNF02 e RNF11, continuam vigentes sem serem consideradas atendidas. Trechos removidos de requisitos vigentes estão na seção 6.1, e cada divergência entre especificação e implementação é descrita somente na seção 6.3, com apontamento no requisito afetado.

Os identificadores nunca são renumerados. Requisitos novos recebem o próximo número livre. A seção 6 lista os itens cancelados e fora de escopo, e a seção 7 traz a matriz de rastreabilidade entre requisitos, regras e cenários Gherkin.

### 1.2. Escopo

**Incluído e implementado (vigente):**

- Cadastro, ativação por e-mail, reenvio de ativação, login, logout, recuperação e redefinição de senha, alteração de nome de usuário e de senha, preferência de conteúdo adulto e exclusão da própria conta.
- Perfis de acesso por conta (Usuário Padrão e Administrador), com perfil ativo por sessão.
- Catálogo público de mangás publicados no Brasil: vitrine de Obras, vitrine de Edições, página do Autor e detalhes de Obra, Edição e Volume, com URLs contextualizadas.
- Controle de conteúdo adulto no catálogo público.
- Painel administrativo para curadoria: Obras, Edições, Volumes, listas de valores (“Gerenciar opções”), contas de usuário e capas internas.

**Futuro (planejado):**

- Estante Digital (coleção pessoal), Lista de Desejos e Calendário de Lançamentos (tela “Checklist”). Hoje existem apenas as telas “Em breve” em `/colecao`, `/desejos` e `/checklist`, a tela de seleção visual de Volumes e os botões “Coleção” e “Lista de Desejos” sem ação na página do Volume. Não há API nem persistência para esses recursos.

**Fora de escopo:**

- Funcionalidades de rede social (seguir usuários, mensagens diretas).
- E-commerce ou venda direta. O único vínculo comercial é o link de afiliado do Volume.
- Aplicativo móvel nativo. A aplicação é web e responsiva.
- Catalogação de mangás não publicados oficialmente no Brasil.
- Avaliações e resenhas de usuários.
- Edições ou volumes digitais.

### 1.3. Terminologia

- **SERS:** Especificação de Requisitos de Software (este documento).
- **Obra:** a propriedade intelectual original, com os metadados do país de origem: títulos, país de origem, tipo de Obra, autoria, editoras originais, pré-publicações, status e anos da publicação original, demografias, gêneros, sinopse, capa e indicação de conteúdo adulto.
- **Edição:** a publicação física licenciada e lançada no Brasil por uma editora brasileira. Uma Obra pode ter várias Edições, identificadas pelo número da edição (1ª, 2ª...). Cada Edição tem editora brasileira, status de publicação no Brasil e metadados físicos opcionais (acabamento, formato e miolo). A Edição não tem capa própria.
- **Volume:** o livro físico unitário que pertence a uma Edição, com número, capa, data de lançamento, preço, páginas, ISBN, link de afiliado e sinopse.
- **Hierarquia editorial:** Obra → Edição → Volume. O foco dos dados editoriais é a publicação brasileira.
- **Número da edição:** inteiro positivo, único na Obra, exibido em forma ordinal (“2ª edição”).
- **Acabamento:** tipo de capa física da Edição (lista “Tipos de capa”).
- **Formato:** dimensões físicas da Edição (lista “Formatos físicos”).
- **Miolo:** tipo de papel interno da Edição (lista “Miolos”); uma Edição pode combinar vários miolos, em ordem. Não confundir com papel de autoria.
- **Papel de autoria (crédito):** função de um autor na Obra: História e Arte, História, Arte, Criador Original, História Original, Ilustrador ou Design de Personagens.
- **Pré-publicação:** revista de serialização em que a Obra foi publicada antes dos volumes (lista “Revistas de serialização”).
- **Lançamento direto:** Obra publicada diretamente em volume, sem pré-publicação.
- **Capa interna:** imagem importada, processada e armazenada na infraestrutura de mídia do CoMangá (Cloudflare R2). É a única capa exibida.
- **Capa derivada da Edição:** a capa exibida para a Edição, que é a capa do seu Volume 1.
- **Visibilidade:** estado “Público” ou “Privado” de Obra, Edição e Volume. Somente registros públicos em toda a hierarquia aparecem no catálogo público.
- **Conteúdo adulto:** Obra marcada como adulta ou associada ao gênero Hentai.
- **Lista administrativa (categoria):** conjunto de valores usados nos formulários e gerenciado em “Gerenciar opções”. As listas são: autores, pré-publicações (revistas de serialização) e editoras originais, usadas no formulário de Obra; e editoras brasileiras, acabamentos (tipos de capa), formatos e miolos, usadas no formulário de Edição.
- **Valor controlado pelo sistema:** tipo de Obra ou gênero. São valores fixos, criados pelas migrations, sem nenhuma gestão pelo Administrador: não aparecem em “Gerenciar opções”.
- **Valor nativo:** valor fixo do sistema, fora das listas administrativas: país de origem, demografia, status de publicação, papel de autoria, precisão da data e moeda.
- **Slug:** identificador textual da Obra na URL, gerado a partir do título em português no cadastro.
- **URL contextualizada:** caminho público que carrega a Obra e a Edição, como `/obras/{slug}/edicao/{editionId}/volume/{volumeId}`.
- **Conta:** registro de acesso de uma pessoa, com e-mail, nome de usuário, senha, data de nascimento e status.
- **Status da conta:** “Pendente” (aguardando ativação), “Ativada” ou “Bloqueada”. O status “Bloqueada” é reconhecido pelo sistema, mas o fluxo que o atribui é planejado (RF0054).
- **Perfil:** conjunto de permissões atribuível a uma conta. Existem dois perfis de sistema: **Usuário Padrão** e **Administrador**.
- **Perfis concedidos:** os perfis que a conta possui. Toda conta possui Usuário Padrão. Administrador é concedido por outro Administrador.
- **Perfil preferido:** o perfil salvo na conta e usado como perfil ativo no próximo login.
- **Perfil ativo:** o perfil em uso em uma sessão. Define o contexto de navegação e a autorização administrativa. Trocar o perfil ativo não concede nem remove perfis.
- **Nível de acesso:** termo legado. Na interface de contas, indica apenas se a conta possui ou não o perfil Administrador. O campo `nivel_acesso` é mantido sincronizado por compatibilidade e não é usado para autorizar.
- **Sessão:** vínculo autenticado entre o navegador e a API, transportado por cookie HttpOnly e guardado no servidor.
- **Estante Digital** (futuro): coleção pessoal de Volumes de um usuário.
- **Lista de Desejos** (futuro): Volumes que o usuário pretende comprar.
- **Calendário de Lançamentos** (futuro): lista mensal de Volumes previstos.
- **Curadoria:** manutenção da precisão das informações do catálogo pelo Administrador.

“Perfil”, “papel de autoria”, “role” e “nível de acesso” não são sinônimos. Neste documento, “perfil” refere-se a permissões da conta. “Papel” refere-se apenas a créditos de autoria. “Role” aparece somente como nome de campo legado na API.

## 2. Descrição Geral

**Visão do Produto:** o CoMangá é uma aplicação web independente de apoio a colecionadores e leitores de mangás no Brasil. Ele não substitui lojas nem editoras. Atua como camada de organização e informação sobre as publicações brasileiras. A principal dependência externa é a disponibilidade de informações públicas sobre os lançamentos. A integração comercial se limita ao link de afiliado de cada Volume, exibido na Web como “Comprar na Amazon”.

**Contexto técnico resumido:** a Web é uma SPA React publicada na Vercel, que repassa `/api` para a API. A API é Express com Prisma, publicada no Render, sobre PostgreSQL hospedado no Neon, com bancos separados para desenvolvimento, testes e implantação. As capas ficam no Cloudflare R2 e os e-mails transacionais são enviados pelo Resend.

### 2.1. Usuários e contexto

- **Visitante:** pessoa sem sessão. Navega pelo catálogo público, cadastra conta, ativa, reenvia a ativação e recupera a senha. Não vê conteúdo adulto.
- **Usuário Padrão:** conta Ativada navegando com o perfil Usuário Padrão ativo. Usa o catálogo público e as configurações da conta. Vê conteúdo adulto somente se for maior de idade e tiver a preferência habilitada. As personas de referência são:
  - **Colecionador Dedicado:** busca eficiência, precisão e controle. Precisará da Estante Digital e da Lista de Desejos (futuro).
  - **Novo Entusiasta:** precisa de interface simples, descoberta, sinopses claras e um guia de lançamentos (futuro).
- **Administrador / Curador de Dados:** conta com o perfil Administrador concedido. Com esse perfil ativo, mantém a integridade e a normalização do catálogo (Obras, Edições e Volumes sem duplicações), as listas administrativas e os perfis das demais contas. Para navegar no catálogo público, troca o perfil ativo para Usuário Padrão.
- **Serviços externos:** Resend (e-mail), Cloudflare R2 (mídia) e a origem remota informada na importação de capa.

## 3. Requisitos Funcionais

### 3.1. Módulo: Autenticação e Perfil de Usuário

#### RF0001 — Cadastrar conta de acesso e disparar e-mail com link de ativação

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0001, RN0002, RN0003, RN0004, RN0005, RN0006, RN0007, RN0020, RN0048
- **Descrição:** o visitante cadastra uma conta informando nome de usuário, e-mail, data de nascimento, senha e confirmação de senha. A data de nascimento deve ser válida e não futura. A conta é criada com status “Pendente”, perfil Usuário Padrão e preferência de conteúdo adulto desativada. O sistema gera um token de ativação válido por 24 horas e envia ao e-mail informado o link de ativação. Se o envio falhar, a conta permanece criada e a resposta orienta o uso do reenvio.

#### RF0002 — Ativar conta de acesso via token de ativação

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0007, RN0008
- **Descrição:** ao abrir o link de ativação (`/activate/{token}`), o sistema altera o status da conta de “Pendente” para “Ativada” e invalida o token. A tela de ativação não exige sessão.

#### RF0003 — Reenviar e-mail com link de ativação da conta de acesso

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Importante
- **Estado:** Vigente
- **RNs associadas:** RN0007, RN0009, RN0010, RN0011
- **Descrição:** o visitante informa o e-mail de uma conta “Pendente” e recebe um novo link de ativação. O sistema gera um novo token de 24 horas, que invalida o anterior.

#### RF0004 — Autenticar credenciais da conta de acesso e retornar sessão de acesso

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0012, RN0013, RN0014, RN0062
- **Descrição:** o visitante informa e-mail e senha. Com credenciais válidas e conta Ativada, o sistema cria uma sessão no servidor e envia ao navegador apenas um identificador opaco em cookie HttpOnly. A sessão começa com o perfil preferido da conta, se ele ainda estiver concedido, ou com Usuário Padrão. A resposta informa nome de usuário, perfis concedidos e perfil ativo. A Web confirma com “Login realizado com sucesso!” e abre a página de perfil. O excesso de tentativas é limitado conforme RNF08. Divergência na implementação: ver seção 6.3.

#### RF0005 — Encerrar sessão de acesso ativa

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0014
- **Descrição:** o usuário encerra a sessão atual. O sistema revoga a sessão no servidor e remove o cookie. A Web exibe somente a confirmação “Sessão encerrada com segurança.” e leva à tela de login, mesmo que a chamada à API falhe, sem exibir em paralelo o aviso de sessão inválida.

#### RF0006 — Solicitar redefinição de senha de acesso via e-mail

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0015, RN0016, RN0017
- **Descrição:** o visitante informa o e-mail e sempre recebe a resposta neutra de RN0015. Somente para conta elegível (RN0016) o sistema gera um token de redefinição válido por 1 hora e envia o link de recuperação, sem revelar na resposta a existência ou o status da conta.

#### RF0007 — Redefinir senha de acesso via token de redefinição

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0002, RN0014, RN0017, RN0018, RN0019, RN0020
- **Descrição:** a partir do link recebido (`/redefinir-senha/{token}`), o visitante informa nova senha e confirmação. Com token válido, o sistema altera a senha, consome o token, encerra todas as sessões da conta e responde “Senha redefinida com sucesso. Faça login novamente.”.

#### RF0008 — Redefinir senha de acesso em sessão ativa

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Importante
- **Estado:** Vigente
- **RNs associadas:** RN0002, RN0014, RN0020, RN0021, RN0073
- **Descrição:** nas configurações avançadas do perfil, o usuário informa senha atual, nova senha e confirmação. O sistema altera a senha da conta identificada pela sessão, mantém autenticada a sessão usada na alteração e encerra as demais sessões da conta. A resposta é “Senha alterada com sucesso! As demais sessões da conta foram encerradas.”. Tentativas com senha atual incorreta são limitadas por conta (RNF08).

#### RF0009 — Consultar dados cadastrais da própria conta

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0022, RN0062
- **Descrição:** o sistema retorna os dados da conta identificada pela sessão: nome de usuário, e-mail, preferência efetiva de conteúdo adulto, indicação de elegibilidade para habilitá-la (18 anos completos), perfis concedidos e perfil ativo. A data de nascimento não é retornada.

#### RF0010 — Alterar nome de usuário em sessão ativa

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Importante
- **Estado:** Vigente
- **RNs associadas:** RN0001, RN0004, RN0022
- **Descrição:** nas configurações avançadas do perfil, o usuário informa um novo nome de usuário. O sistema altera o nome da conta identificada pela sessão. Não há período de carência nem histórico de nomes. A Web recusa repetir o nome atual sem consultar a API.

#### RF0011 — Alterar preferência de exibição de conteúdo adulto

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0022, RN0048
- **Descrição:** o usuário ativa ou desativa a preferência de conteúdo adulto da própria conta. A ativação só é aceita quando a data de nascimento indica 18 anos completos. A Web só exibe o controle quando a conta é elegível e o perfil ativo não é Administrador.

#### RF0012 — Excluir permanentemente conta de acesso e dados vinculados

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0014, RN0021, RN0022, RN0063
- **Descrição:** nas configurações avançadas do perfil, o usuário confirma a senha atual e exclui permanentemente a própria conta e os dados que dependem exclusivamente dela (sessões, perfis concedidos e tokens). A exclusão encerra as sessões e não aceita identificador de conta vindo do cliente. A conta do último Administrador efetivo não pode ser excluída.

#### RF0052 — Alternar perfil ativo da sessão

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0062, RN0064, RN0065
- **Descrição:** a conta com mais de um perfil concedido escolhe, na página de perfil, qual perfil usar na sessão atual. O sistema aceita apenas perfis concedidos à conta, altera o perfil ativo somente desta sessão, salva a escolha como perfil preferido e responde “Perfil ativo atualizado com sucesso!”. A Web confirma com “Perfil ativo alterado para {perfil}.” e atualiza o contexto sem recarregar a página. Contas só com Usuário Padrão não veem o seletor.

#### RF0053 — Direcionar o acesso às páginas conforme sessão e perfil ativo

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0014, RN0064, RN0065
- **Descrição:** a Web direciona cada pessoa conforme a sessão e o perfil ativo. Páginas de visitante enviam usuários autenticados ao perfil. Páginas protegidas enviam visitantes ao login. Páginas administrativas exigem o perfil Administrador ativo. Páginas públicas do catálogo não são exibidas com o perfil Administrador ativo. A API revalida a autorização em cada rota protegida.

### 3.2. Módulo: Administração

#### RF0013 — Cadastrar nova Obra no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0023, RN0024, RN0025, RN0026, RN0027, RN0066, RN0067, RN0068
- **Descrição:** o Administrador cadastra uma Obra em “Novo mangá” (`/admin/novo-manga`), num formulário em quatro etapas:
  1. **Identificação:** título em português, título original, título romanizado, país de origem e tipo de Obra.
  2. **Autoria:** um ou mais autores, cada um com um ou mais papéis de autoria.
  3. **Publicação original e classificação:** editoras originais (em ordem definida), status da publicação original, anos de início e fim, lançamento direto, pré-publicações (em ordem definida), demografias, gêneros e indicação de conteúdo adulto.
  4. **Capa e sinopse:** capa importada por URL (RF0050), exibida antes da sinopse.

  A Obra é gravada numa única transação, com visibilidade “Privado” e slug único gerado a partir do título. Depois do cadastro, a Web oferece as próximas ações (`/admin/pos-cadastro`).

#### RF0014 — Alterar dados de uma Obra específica no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0023, RN0024, RN0025, RN0026, RN0060, RN0066, RN0067, RN0068
- **Descrição:** o Administrador altera parcial ou totalmente os dados de uma Obra (`/admin/gerenciar-mangas/obras/{slug}/editar`). O sistema persiste somente os campos enviados e preserva os demais. Listas enviadas (autores, gêneros, demografias, editoras originais, pré-publicações) substituem integralmente as anteriores. A troca de capa só ocorre com uma nova capa interna válida. O slug não muda quando o título é alterado.

#### RF0015 — Excluir uma Obra específica do banco de dados

- **Módulo:** Administração
- **Prioridade:** Importante
- **Estado:** Vigente
- **RNs associadas:** RN0028, RN0029
- **Descrição:** o Administrador exclui permanentemente uma Obra privada e sem Edições, após confirmação. A capa interna que fica sem uso é descartada.

#### RF0016 — Consultar coleção de Obras cadastradas no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0061
- **Descrição:** em “Gerenciar mangás” (`/admin/gerenciar-mangas`), o sistema lista as Obras de forma paginada (até 50 por página), em lista ou grade. Cada Obra mostra capa, título, autores ordenados pelo crédito, país de origem, tipo de Obra, quantidade de Edições calculada e visibilidade. O título original está disponível na API, mas a Web não o exibe na lista. A busca textual parcial é pelo título em português. Os filtros combináveis são tipo de Obra, país de origem e visibilidade. Na Web, a ordenação é por título, crescente (padrão) ou decrescente. A ordenação por autor, país, tipo, quantidade de Edições ou visibilidade está disponível na API.

#### RF0017 — Consultar dados de uma Obra específica do banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** N/A
- **Descrição:** o sistema retorna todos os dados de uma Obra, localizada pelo identificador ou pelo slug usado nas rotas administrativas, refletindo o estado mais recente do registro.

#### RF0018 — Alterar status de visibilidade de uma Obra específica do banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0030
- **Descrição:** o Administrador alterna a visibilidade de uma Obra entre “Público” e “Privado”. Tornar a Obra pública não publica suas Edições. Tornar a Obra privada exige que nenhuma Edição vinculada esteja pública.

#### RF0019 — Cadastrar nova Edição vinculada a uma Obra no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0031, RN0032, RN0033, RN0034, RN0069
- **Descrição:** o Administrador cadastra uma Edição vinculada a uma Obra existente (`/admin/gerenciar-mangas/obras/{slug}/edicoes/nova`). Informa número da edição, editora brasileira e status de publicação no Brasil (obrigatórios) e, se conhecidos, acabamento, formato e um ou mais miolos. A Edição nasce “Privado” e não recebe capa própria: sua capa é derivada do Volume 1.

#### RF0020 — Alterar dados de uma Edição específica no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0031, RN0032, RN0033
- **Descrição:** o Administrador altera parcial ou totalmente os dados de uma Edição (`.../edicoes/{editionId}/editar`). O sistema persiste somente os campos enviados. Acabamento e formato podem voltar a nulo; os miolos podem ser removidos enviando uma lista vazia. A lista enviada substitui integralmente a anterior, e sua omissão preserva os vínculos.

#### RF0021 — Excluir uma Edição específica do banco de dados

- **Módulo:** Administração
- **Prioridade:** Importante
- **Estado:** Vigente
- **RNs associadas:** RN0035, RN0036
- **Descrição:** o Administrador exclui permanentemente uma Edição privada e sem Volumes, após confirmação.

#### RF0022 — Consultar coleção de Edições vinculadas a uma Obra no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0051, RN0061, RN0069
- **Descrição:** em “Gerenciar edições” (`/admin/gerenciar-mangas/obras/{slug}/edicoes`), o sistema mostra a prévia da Obra e lista, de forma paginada (até 50 por página), suas Edições. Cada Edição mostra capa derivada do Volume 1 (ou “Sem capa (cadastre o Volume 1)”), número em formato ordinal, editora brasileira, quantidade de Volumes calculada e visibilidade. Acabamento, formato, miolo e status no Brasil estão disponíveis na API, mas a Web não os exibe nessa lista. A ordenação padrão é decrescente pelo número da edição.

#### RF0023 — Consultar dados de uma Edição específica do banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** N/A
- **Descrição:** o sistema retorna todos os dados de uma Edição pelo identificador, com a Obra vinculada e a capa derivada, refletindo o estado mais recente do registro.

#### RF0024 — Alterar status de visibilidade de uma Edição específica do banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0037, RN0040, RN0042, RN0069
- **Descrição:** o Administrador alterna a visibilidade de uma Edição entre “Público” e “Privado”. A publicação exige Obra pública e Volume 1 com capa interna. O novo status é aplicado a todos os Volumes da Edição na mesma transação. O bloqueio por Volumes em acervos pessoais (RN0042) depende da Estante Digital e ainda não se aplica.

#### RF0025 — Cadastrar novo Volume vinculado a uma Edição no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0038, RN0039, RN0040
- **Descrição:** o Administrador cadastra um Volume vinculado a uma Edição (`.../volumes/novo`). Informa número (a partir de zero), capa interna, precisão e data de lançamento e, opcionalmente, indicação de volume único, páginas, moeda e preço de capa (R$ por padrão), ISBN-10, ISBN-13, link de afiliado e sinopse. O Volume herda a visibilidade vigente da Edição.

#### RF0026 — Alterar dados de um Volume específico no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0038, RN0039, RN0060, RN0069
- **Descrição:** o Administrador altera parcial ou totalmente os dados de um Volume (`.../volumes/{volumeId}/editar`). O sistema persiste somente os campos enviados, valida a data resultante e preserva a capa quando não há substituta.

#### RF0027 — Excluir um Volume específico do banco de dados

- **Módulo:** Administração
- **Prioridade:** Importante
- **Estado:** Vigente
- **RNs associadas:** RN0041
- **Descrição:** o Administrador exclui permanentemente um Volume privado, após confirmação. A capa interna que fica sem uso é descartada. Se o Volume excluído era o Volume 1, a Edição fica sem capa derivada.

#### RF0028 — Consultar coleção de Volumes vinculados a uma Edição no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0061
- **Descrição:** em “Gerenciar volumes” (`/admin/gerenciar-mangas/obras/{slug}/edicoes/{editionId}/volumes`), o sistema lista os Volumes da Edição de forma paginada (até 50 por página), em lista ou grade. Cada Volume mostra capa, número ou indicação de volume único, data de lançamento conforme a precisão e visibilidade. A ordenação padrão é crescente pelo número. Os breadcrumbs identificam a Obra e a Edição.

#### RF0029 — Consultar dados de um Volume específico do banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** N/A
- **Descrição:** o sistema retorna todos os dados de um Volume pelo identificador (`.../volumes/{volumeId}`), refletindo o estado mais recente do registro.

#### RF0030 — Cadastrar novo valor em uma lista de valores pré-cadastrados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0043, RN0066
- **Descrição:** em “Gerenciar opções” (`/admin/opcoes`), o Administrador inclui um ou mais valores em uma lista gerenciável: autores, pré-publicações ou editoras originais (formulário de Obra), ou editoras brasileiras, acabamentos, formatos ou miolos (formulário de Edição). Vários valores podem ser informados de uma vez, separados por vírgula. Em formatos, a vírgula faz parte do valor (por exemplo, “13,5 x 20,5 cm”). Divergência na implementação (miolos): ver seção 6.3. Autores, pré-publicações e editoras originais exigem ao menos um país de origem relacionado.

#### RF0031 — Alterar um valor específico de uma lista de valores pré-cadastrados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0043, RN0066
- **Descrição:** o Administrador altera o texto de um valor de lista gerenciável e, quando aplicável, os países relacionados, sem perder os vínculos já existentes. A edição é feita na própria linha do valor. A API também permite ativar e desativar valores dessas listas; por decisão, essa ação existe só na API e a Web não a oferece. Valores inativos não aparecem nos formulários nem nos filtros do catálogo público.

#### RF0032 — Excluir um valor específico de uma lista de valores pré-cadastrados

- **Módulo:** Administração
- **Prioridade:** Importante
- **Estado:** Vigente
- **RNs associadas:** RN0044, RN0066
- **Descrição:** o Administrador exclui permanentemente um valor sem vínculos de uma lista gerenciável, após confirmação.

#### RF0033 — Consultar coleção de valores de listas pré-cadastradas

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0066
- **Descrição:** em “Gerenciar opções”, o Administrador escolhe o formulário (Obra ou Edição) e a lista, e o sistema exibe os valores de forma paginada e pesquisável, com texto e, quando aplicável, os países relacionados. Listas com país relacionado mostram 5 valores por página e as demais mostram 6. A ordenação é alfabética crescente ou decrescente. A API também permite consultar tipos de Obra, gêneros e países de origem, que alimentam os formulários, mas essas categorias não são oferecidas para gestão.

#### RF0034 — Consultar coleção de contas de usuário cadastradas no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0062
- **Descrição:** em `/admin/users`, o sistema lista as contas de forma paginada (8 por página na Web). Cada conta mostra nome de usuário, e-mail, nível de acesso derivado (Administrador quando possui esse perfil, senão Usuário Padrão) e status (Pendente, Ativada ou Bloqueada). O identificador e a lista de perfis concedidos estão disponíveis na API, mas a Web não os exibe na tabela. Há busca textual parcial por nome de usuário ou e-mail, filtros combináveis por nível de acesso e status e ordenação crescente ou decrescente pelo nome de usuário. A linha da própria conta não permite alteração.

#### RF0035 — Conceder ou remover perfil Administrador

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0045, RN0062, RN0063
- **Descrição:** o Administrador concede ou remove o perfil Administrador de **outra** conta. Conceder mantém o perfil Usuário Padrão, define Administrador como perfil preferido da conta alvo e não altera o perfil ativo das sessões já abertas. Remover mantém Usuário Padrão, troca para Usuário Padrão as sessões abertas que estavam com Administrador ativo e redefine o perfil preferido. A própria conta e a remoção do último Administrador efetivo são bloqueadas.

#### RF0054 — Bloquear e desbloquear conta de usuário

- **Módulo:** Administração
- **Prioridade:** Importante
- **Estado:** Futuro (planejado)
- **RNs associadas:** RN0013
- **Descrição:** o Administrador poderá bloquear e desbloquear outra conta. O status “Bloqueada” já é reconhecido pelo sistema: impede o login (RN0013), o reenvio de ativação e a recuperação de senha, e aparece no filtro de contas. Falta o fluxo que atribui e retira esse status. Quem pode bloquear, o efeito sobre as sessões abertas e o desbloqueio serão especificados na implementação.

#### RF0050 — Importar e gerenciar capa interna por URL

- **Módulo:** Administração
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0058, RN0059, RN0060, RN0061
- **Descrição:** nos formulários de Obra e de Volume, o Administrador informa a URL HTTPS de uma imagem. A API baixa a imagem, valida, processa (variantes WebP em proporção 2:3) e armazena no R2, criando uma capa interna pendente. A capa só é associada quando o formulário é salvo. Uma capa pendente pode ser descartada antes da associação. Na alteração, a nova capa substitui a anterior de forma atômica, e a capa anterior que fica sem uso é descartada. Não é possível remover uma capa associada sem substituí-la. A importação é genérica, sem regras por provedor.

### 3.3. Módulo: Catálogo Público e Busca

#### RF0036 — Consultar vitrine pública de Obras

- **Módulo:** Catálogo Público e Busca
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0046, RN0047, RN0048, RN0049, RN0065
- **Descrição:** em `/pesquisa` (aba Obras), visitantes e Usuários Padrão consultam, de forma paginada, as Obras públicas. A busca textual parcial considera título, título original, título romanizado e nome do autor. Os filtros são tipo de Obra (restrito aos tipos compatíveis com o país selecionado), país de origem, demografias e gêneros (múltiplos, por interseção), editora original, revista de serialização, status da publicação original e anos de início e fim. Na Web, a ordenação é sempre por título, crescente (padrão) ou decrescente. A ordenação por título original ou por data de cadastro está disponível na API. Busca, filtros, ordenação e página ficam registrados na URL. As opções de filtro vêm do sistema e omitem valores inativos e o gênero restrito para quem não pode ver conteúdo adulto.

#### RF0037 — Consultar vitrine pública de Edições

- **Módulo:** Catálogo Público e Busca
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0046, RN0047, RN0048, RN0049, RN0051, RN0065, RN0069
- **Descrição:** em `/pesquisa` (aba Edições), o sistema lista, de forma paginada, as Edições públicas de Obras públicas. A busca considera título, título original, título romanizado e autor da Obra. Os filtros são editora brasileira, formato, acabamento, número da edição, status no Brasil e anos de início e fim da publicação brasileira (derivados das datas dos Volumes públicos). Na Web, a ordenação é sempre por título da Obra, crescente (padrão) ou decrescente. A ordenação por número da edição ou por data de cadastro está disponível na API. Cada Edição mostra capa derivada, título da Obra e “Nª edição · editora”. Autores, formato, acabamento e total de Volumes públicos estão disponíveis na API, mas a Web não os exibe no cartão. Ao alternar entre as abas, o termo e a ordenação compatível são preservados.

#### RF0051 — Consultar Obras públicas de um Autor

- **Módulo:** Catálogo Público e Busca
- **Prioridade:** Importante
- **Estado:** Vigente
- **RNs associadas:** RN0046, RN0048, RN0065
- **Descrição:** em `/autores/{authorId}`, o sistema exibe o nome do autor e, de forma paginada, as Obras públicas em que ele tem crédito, na mesma grade da vitrine, em ordem crescente de título. A Web não oferece outra ordenação. A ordem decrescente e a ordenação por título original ou por data de cadastro estão disponíveis na API. Autor inexistente ou identificador inválido resulta em “Autor não encontrado.” e há estado vazio com retorno ao catálogo.

### 3.4. Módulo: Catálogo Público e Navegação Profunda

#### RF0038 — Consultar detalhes públicos de uma Obra

- **Módulo:** Catálogo Público e Navegação Profunda
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0046, RN0048, RN0050, RN0061, RN0065, RN0071, RN0072
- **Descrição:** em `/obras/{slug}`, visitantes e Usuários Padrão veem a ficha pública da Obra: título, título original e título romanizado, capa, país de origem, tipo de Obra, autores com seus papéis (ordenados pelo crédito), editoras originais e pré-publicações, demografias e gêneros, status e anos da publicação original, sinopse e Edições públicas (RF0039). A indicação de lançamento direto está disponível na API, mas a Web não a exibe. Metadados opcionais ausentes são omitidos (RN0072). Divergência na implementação: ver seção 6.3. O enriquecimento com dados pessoais de posse e desejo (RN0050) é futuro.

#### RF0039 — Consultar Edições públicas vinculadas a uma Obra

- **Módulo:** Catálogo Público e Navegação Profunda
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0046, RN0048, RN0050, RN0051, RN0061, RN0065, RN0069, RN0072
- **Descrição:** na ficha da Obra, cada Edição pública aparece com número da edição, editora brasileira, status no Brasil acompanhado do total de Volumes públicos e prévia dos três primeiros Volumes públicos. A capa derivada, o formato, o acabamento e o miolo de cada Edição estão disponíveis na API, mas a Web não os exibe nessa lista. Edições e Volumes privados são omitidos.

#### RF0040 — Consultar detalhes públicos de uma Edição

- **Módulo:** Catálogo Público e Navegação Profunda
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0046, RN0048, RN0050, RN0051, RN0061, RN0065, RN0069, RN0071, RN0072
- **Descrição:** em `/obras/{slug}/edicao/{editionId}`, o sistema exibe o título da Obra, a capa derivada, a editora brasileira, o número da edição, o status no Brasil, o formato, o acabamento e os miolos quando informados (exibidos empilhados), os anos de início e fim da publicação no Brasil (derivados dos Volumes públicos, com o fim apenas para Edição “Completa”), o total de Volumes públicos e a listagem paginada desses Volumes, com página registrada na URL. O título original e os autores da Obra estão disponíveis na API, mas a Web não os exibe nessa página. Metadados opcionais ausentes são omitidos (RN0072). Divergência na implementação: ver seção 6.3. Há navegação para a Obra e para cada Volume por URLs contextualizadas.

#### RF0041 — Consultar detalhes públicos de um Volume

- **Módulo:** Catálogo Público e Navegação Profunda
- **Prioridade:** Essencial
- **Estado:** Vigente
- **RNs associadas:** RN0046, RN0048, RN0050, RN0061, RN0065, RN0070, RN0071, RN0072
- **Descrição:** em `/obras/{slug}/edicao/{editionId}/volume/{volumeId}`, o sistema exibe Obra e Edição vinculadas, capa, número ou indicação de volume único, data de lançamento conforme a precisão, preço com moeda, número de páginas, ISBN-10, ISBN-13 e sinopse. Há retorno contextual para a Edição e para a Obra e navegação para os Volumes públicos anterior e seguinte da mesma Edição. Com link de afiliado, o botão “Comprar na Amazon” abre a loja em nova aba. Sem link, o botão aparece desabilitado com a marca “Indisponível”. Os botões “Coleção” e “Lista de Desejos” ainda não têm ação (Estante e Lista de Desejos são futuras).

### 3.5. Módulo: Catálogo Público e Calendário de Lançamentos

#### RF0042 — Consultar calendário público de lançamentos

- **Módulo:** Catálogo Público e Calendário de Lançamentos
- **Prioridade:** Essencial
- **Estado:** Futuro (planejado)
- **RNs associadas:** RN0046, RN0048, RN0052
- **Descrição:** o sistema deverá permitir que visitantes e usuários consultem os Volumes públicos previstos para determinado mês e ano, desde que a precisão da data permita identificar o mês, exibindo Obra, Edição, número e capa do Volume, data de lançamento e editora brasileira, com filtro por editora e busca por título ou autor. Sem mês e ano informados, deverá usar o período vigente. Na Web existe apenas a tela “Checklist” (`/checklist`), com o aviso “Em breve: Acompanhe seus lançamentos mensais.”

### 3.6. Módulo: Estante Digital e Lista de Desejos

Todos os requisitos desta seção estão no estado **Futuro (planejado)**. A Web tem apenas as telas “Em breve” `/colecao` e `/desejos`, a tela de seleção visual de Volumes (`/obras/{slug}/edicao/{editionId}/selecionar/{estante|desejos}`), que não grava nada, e os botões sem ação na página do Volume. Não há tabelas nem rotas de API para acervo pessoal.

#### RF0043 — Registrar posse individual de Volume

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **Estado:** Futuro (planejado)
- **RNs associadas:** RN0053, RN0054, RN0055, RN0056
- **Descrição:** o usuário autenticado deverá registrar um Volume público como adquirido, criando o vínculo na Estante Digital com estado de leitura “não lido” e removendo o mesmo Volume da Lista de Desejos, se presente.

#### RF0044 — Atualizar estado de leitura ou remover posse individual de Volume

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **Estado:** Futuro (planejado)
- **RNs associadas:** RN0053, RN0055
- **Descrição:** o usuário deverá marcar como lido ou não lido um Volume da Estante Digital e remover o vínculo de posse, sem excluir o Volume do catálogo. Ao remover a posse, o estado de leitura é removido junto.

#### RF0045 — Sincronizar registros de posse em lote

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **Estado:** Futuro (planejado)
- **RNs associadas:** RN0053, RN0054, RN0055, RN0056
- **Descrição:** a partir de uma Edição, o usuário deverá adicionar e remover vários Volumes da Estante Digital numa única operação transacional. Os adicionados saem da Lista de Desejos e iniciam “não lidos”. Os removidos eliminam seus estados de leitura. A seleção visual de Volumes já existe na Web, sem gravação.

#### RF0046 — Consultar registros da Estante Digital

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **Estado:** Futuro (planejado)
- **RNs associadas:** RN0053, RN0055, RN0056
- **Descrição:** o usuário deverá consultar os Volumes da Estante agrupados por Obra e Edição, com a progressão em relação aos Volumes públicos de cada Edição e o estado “lido” ou “não lido” de cada vínculo.

#### RF0047 — Registrar intenção de compra

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **Estado:** Futuro (planejado)
- **RNs associadas:** RN0053, RN0055, RN0056, RN0057
- **Descrição:** o usuário deverá adicionar um Volume público à Lista de Desejos, desde que ainda não o possua na Estante Digital.

#### RF0048 — Remover intenção de compra

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **Estado:** Futuro (planejado)
- **RNs associadas:** RN0053, RN0055
- **Descrição:** o usuário deverá remover um Volume da Lista de Desejos, sem excluí-lo do catálogo.

#### RF0049 — Consultar Lista de Desejos

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **Estado:** Futuro (planejado)
- **RNs associadas:** RN0053, RN0055, RN0056
- **Descrição:** o usuário deverá consultar os Volumes da Lista de Desejos com seus dados públicos essenciais. Registros que deixarem de estar públicos em qualquer nível da hierarquia serão omitidos enquanto permanecerem indisponíveis.

## 4. Regras de Negócio

### 4.1. Conta, sessão e perfis

#### RN0001 — Política de formatação de Username da conta de acesso

- **Estado:** Vigente
- **RFs dependentes:** RF0001, RF0010
- **Descrição:** o nome de usuário deve ter de 3 a 20 caracteres e conter apenas letras sem acento, números e sublinhado (`_`), sem espaços. A mesma regra vale no cadastro e na alteração. A violação é recusada com: "Utilize entre 3 e 20 caracteres, sem espaços, acentos ou caracteres especiais."

#### RN0002 — Política de força da senha da conta de acesso

- **Estado:** Vigente
- **RFs dependentes:** RF0001, RF0007, RF0008
- **Descrição:** a senha deve ter no mínimo 8 caracteres, com ao menos uma letra maiúscula, uma minúscula, um número e um caractere especial. A violação é recusada com: "Utilize no mínimo 8 caracteres, incluindo pelo menos uma letra maiúscula, uma minúscula, um número e um caractere especial." A senha também não pode ultrapassar 72 bytes em UTF-8 (limite do algoritmo de hash). Nesse caso, a mensagem é: "A senha deve ter no máximo 72 bytes em UTF-8; acentos e emojis podem ocupar mais de um byte."

#### RN0003 — Prevenção de duplicidade de e-mails de contas de acesso

- **Estado:** Vigente
- **RFs dependentes:** RF0001
- **Descrição:** o e-mail é único entre todas as contas, qualquer que seja o status. O cadastro com e-mail já usado é recusado com: "Este endereço de e-mail já está em uso. Tente fazer login ou recuperar sua senha."

#### RN0004 — Prevenção de duplicidade de nomes de usuário

- **Estado:** Vigente
- **RFs dependentes:** RF0001, RF0010
- **Descrição:** o nome de usuário é único. O cadastro ou a alteração para um nome já usado por outra conta é recusado com: "Este nome de usuário não está disponível. Por favor, escolha outro."

#### RN0005 — Atribuição compulsória de nível de acesso padrão a novas contas

- **Estado:** Vigente
- **RFs dependentes:** RF0001
- **Descrição:** toda nova conta recebe compulsoriamente o perfil Usuário Padrão, e somente ele. O perfil Administrador só pode ser concedido por outro Administrador (RF0035).

#### RN0006 — Atribuição compulsória da preferência não exibição de conteúdo adulto a novas contas

- **Estado:** Vigente
- **RFs dependentes:** RF0001
- **Descrição:** toda nova conta nasce com a preferência de conteúdo adulto desativada.

#### RN0007 — Tempo de expiração do token de ativação da conta de acesso

- **Estado:** Vigente
- **RFs dependentes:** RF0001, RF0002, RF0003
- **Descrição:** o token de ativação expira 24 horas após a geração. O uso de token expirado é recusado com: "Este link de ativação expirou. Solicite um novo e-mail de ativação."

#### RN0008 — Invalidação por unicidade do token de ativação

- **Estado:** Vigente
- **RFs dependentes:** RF0002
- **Descrição:** o token de ativação é invalidado no primeiro uso bem-sucedido. Nova tentativa, ou token inexistente, é recusada com: "Link de ativação inválido!"

#### RN0009 — Recusa do reenvio do link de ativação a contas não existentes

- **Estado:** Vigente
- **RFs dependentes:** RF0003
- **Descrição:** o reenvio para e-mail não cadastrado é recusado com: "Endereço de e-mail não cadastrado"

#### RN0010 — Recusa do reenvio do link de ativação a contas ativadas

- **Estado:** Vigente
- **RFs dependentes:** RF0003
- **Descrição:** o reenvio para conta “Ativada” é recusado com: "Este endereço de e-mail pertence a uma conta ativada." Para conta “Bloqueada”, a recusa é: "Somente contas pendentes podem solicitar um novo link de ativação."

#### RN0011 — Invalidação por sobreposição do novo token de ativação

- **Estado:** Vigente
- **RFs dependentes:** RF0003
- **Descrição:** um novo token de ativação invalida o anterior. O uso do token antigo é recusado com: "Link de ativação inválido!"

#### RN0012 — Bloqueio de sessão por credenciais inválidas

- **Estado:** Vigente
- **RFs dependentes:** RF0004
- **Descrição:** e-mail ou senha que não correspondam a uma conta são recusados com: "Credenciais inválidas!"

#### RN0013 — Bloqueio de sessão por status de acesso pendente

- **Estado:** Vigente
- **RFs dependentes:** RF0004, RF0054
- **Descrição:** o login em conta “Pendente” é recusado com: "Conta de acesso pendente. Ative a conta com o e-mail de verificação enviado anteriormente." O login em conta “Bloqueada” é recusado com: "Esta conta foi bloqueada por razões de segurança." Sessões de contas que deixem de estar “Ativada” não autorizam rotas protegidas.

#### RN0014 — Política de persistência de sessão de acesso

- **Estado:** Vigente
- **RFs dependentes:** RF0004, RF0005, RF0007, RF0008, RF0012, RF0053
- **Descrição:** cada login cria uma sessão no servidor com identificador opaco e aleatório. Apenas o resumo criptográfico do identificador é armazenado, e o valor original vai ao navegador por cookie HttpOnly. A sessão vale enquanto existir, não estiver revogada e pertencer a uma conta Ativada. Não há expiração por tempo. A sessão é encerrada:
  - no logout (sessão atual);
  - na redefinição de senha por token (todas as sessões da conta);
  - na alteração de senha autenticada (todas, exceto a sessão usada na alteração);
  - na exclusão da conta (todas).

  Rota protegida sem sessão válida responde: "Sua sessão é inválida ou foi encerrada. Por favor, faça login novamente." A Web exibe a mesma mensagem ao enviar um visitante de uma página protegida para o login.

#### RN0015 — Resposta neutra à solicitação de redefinição de senha

- **Estado:** Vigente
- **RFs dependentes:** RF0006
- **Descrição:** para e-mail com formato válido e dentro do limite de solicitações, a resposta é sempre sucesso com: "Se houver uma conta apta para este e-mail, enviaremos as instruções de recuperação.", independentemente da existência da conta, do seu status, de falha no envio do e-mail e do intervalo mínimo entre solicitações. A resposta é enviada antes da consulta à conta, para não revelar elegibilidade pelo tempo de resposta. E-mail inexistente não gera token nem mensagem. Uma nova emissão para a mesma conta só ocorre 60 segundos após a anterior; antes disso, a solicitação é ignorada em silêncio. E-mail com formato inválido é recusado com "Informe um e-mail válido.". O excesso de solicitações por IP é recusado conforme RNF08. Falhas técnicas são registradas sem senhas, tokens ou credenciais.

#### RN0016 — Elegibilidade da conta para recuperação de senha

- **Estado:** Vigente
- **RFs dependentes:** RF0006
- **Descrição:** somente conta “Ativada” recebe token e e-mail de recuperação. Contas “Pendente” ou “Bloqueada” recebem a mesma resposta neutra, sem token nem e-mail. A recuperação não ativa nem desbloqueia contas.

#### RN0017 — Tempo de expiração do token de redefinição de senha de acesso

- **Estado:** Vigente
- **RFs dependentes:** RF0006, RF0007
- **Descrição:** o token de redefinição expira 1 hora após a geração (ao atingir a expiração, já é inválido). É aleatório, armazenado apenas como resumo criptográfico e separado dos tokens de ativação. O valor original existe somente no link enviado, nunca em logs ou respostas. O uso de token expirado é recusado com: "Este link de redefinição expirou. Solicite a redefinição novamente."

#### RN0018 — Invalidação por unicidade do token de redefinição

- **Estado:** Vigente
- **RFs dependentes:** RF0007
- **Descrição:** o token de redefinição é invalidado no primeiro uso bem-sucedido. Nova tentativa, token inexistente ou com formato inválido é recusado com: "Link de redefinição inválido!"

#### RN0019 — Invalidação por sobreposição do novo token de redefinição

- **Estado:** Vigente
- **RFs dependentes:** RF0007
- **Descrição:** um novo token de redefinição invalida os anteriores ainda não usados da conta. O uso de token sobreposto é recusado com: "Link de redefinição inválido!"

#### RN0020 — Recusa de transação por divergência em confirmação de senha

- **Estado:** Vigente
- **RFs dependentes:** RF0001, RF0007, RF0008
- **Descrição:** quando senha (ou nova senha) e confirmação diferem, a operação é recusada com: "Divergência nos valores da senha e confirmação de senha!" Na redefinição por token (RF0007), a Web recusa antes de consultar a API, com "As senhas não conferem." no campo de confirmação.

#### RN0021 — Recusa de transação por divergência de credencial vigente

- **Estado:** Vigente
- **RFs dependentes:** RF0008, RF0012
- **Descrição:** a alteração de senha autenticada e a exclusão da conta exigem a senha atual. Senha atual incorreta é recusada com: "Senha atual incorreta!"

#### RN0073 — Nova senha diferente da atual

- **Estado:** Vigente
- **RFs dependentes:** RF0008
- **Descrição:** na alteração de senha autenticada, a nova senha não pode ser igual à senha atual. A tentativa é recusada com "A nova senha deve ser diferente da senha atual.", e a Web a recusa antes de consultar a API. Não há histórico de senhas anteriores nem essa verificação na redefinição por e-mail.

#### RN0022 — Política de isolamento e privacidade de manipulação de dados do perfil

- **Estado:** Vigente
- **RFs dependentes:** RF0009, RF0010, RF0011, RF0012
- **Descrição:** consulta, alteração e exclusão dos dados da própria conta usam exclusivamente a identidade da sessão validada. As rotas da própria conta (`/api/users/me...`) não recebem identificador de conta pela URL e ignoram qualquer identificador enviado no corpo, de modo que só afetam a conta da sessão. Operações sobre outras contas existem apenas nas rotas administrativas e exigem o perfil Administrador ativo (RN0064).

#### RN0062 — Perfis concedidos, perfil preferido e perfil ativo

- **Estado:** Vigente
- **RFs dependentes:** RF0004, RF0009, RF0034, RF0035, RF0052
- **Descrição:** uma conta pode ter mais de um perfil concedido: Usuário Padrão (sempre presente) e, opcionalmente, Administrador. Cada sessão tem um perfil ativo, que precisa estar entre os perfis concedidos. Caso contrário, vale Usuário Padrão. O perfil ativo de uma sessão não altera o das outras sessões abertas. A troca do perfil ativo grava o perfil preferido da conta, usado como perfil ativo no próximo login se ainda estiver concedido. Trocar o perfil ativo não concede nem remove perfis. Pedir um perfil inexistente é recusado com "Perfil de acesso inexistente." e pedir um perfil não concedido, com "Acesso negado: sua conta não possui este perfil de acesso.". Quando o perfil Administrador é removido, as sessões abertas com ele ativo passam a Usuário Padrão e deixam de ter acesso administrativo.

#### RN0063 — Proteção do último Administrador

- **Estado:** Vigente
- **RFs dependentes:** RF0012, RF0035
- **Descrição:** o sistema não pode ficar sem nenhum Administrador efetivo (conta Ativada com o perfil Administrador). A remoção do perfil do último Administrador e a exclusão da conta do último Administrador são recusadas com: "Operação bloqueada: o sistema ficaria sem nenhum administrador ativo." Operações concorrentes são serializadas para que a regra valha mesmo com pedidos simultâneos.

#### RN0064 — Autorização administrativa pelo perfil ativo

- **Estado:** Vigente
- **RFs dependentes:** RF0013 a RF0035, RF0050, RF0052, RF0053
- **Descrição:** todas as rotas administrativas da API exigem sessão válida com o perfil Administrador ativo e ainda concedido à conta. Usuário Padrão, ou conta com Administrador concedido mas Usuário Padrão ativo, recebe recusa com a mensagem "Acesso negado: voce nao possui permissao para executar esta operacao." Divergência na implementação: ver seção 6.3. Na Web, quem acessa uma página administrativa sem o perfil Administrador ativo é levado ao próprio perfil com o aviso "Acesso negado: Você não tem permissão para acessar esta área."

#### RN0065 — Separação de páginas por sessão e perfil ativo na Web

- **Estado:** Vigente
- **RFs dependentes:** RF0036, RF0037, RF0038, RF0039, RF0040, RF0041, RF0051, RF0052, RF0053
- **Descrição:**
  - **Páginas de visitante** (`/entrar`, `/cadastrar`, `/reenvio`, `/recuperar-senha`, `/redefinir-senha/{token}`): usuário autenticado é levado ao próprio perfil (`/perfil/{nome de usuário}`). A raiz `/` leva a `/entrar`. A ativação (`/activate/{token}`) não tem restrição.
  - **Páginas protegidas** (`/perfil`, `/perfil/{nome de usuário}` e páginas administrativas): visitante é levado a `/entrar` com o aviso de sessão inválida (RN0014).
  - **Páginas administrativas** (`/admin/...`): exigem o perfil Administrador ativo (RN0064).
  - **Páginas públicas do catálogo** (`/pesquisa`, `/autores/{id}`, `/obras/{slug}` e suas rotas de Edição, seleção e Volume): com o perfil Administrador ativo, a pessoa é levada ao próprio perfil com o aviso exato "Mude o perfil para usuário padrão para acessar essa página." Visitantes e Usuários Padrão acessam normalmente.

  As telas “Em breve” (`/colecao`, `/checklist`, `/desejos`) não têm essa proteção. Divergência na implementação (rotas `/colecao/{slug}/edicao/{editionId}` e `/colecao/{slug}/edicao/{editionId}/selecionar/{modo}`): ver seção 6.3.

### 4.2. Obra, autoria e classificações

#### RN0023 — Política de multiplicidade de vínculos da Obra

- **Estado:** Vigente
- **RFs dependentes:** RF0013, RF0014
- **Descrição:** uma Obra pode ter vários autores, papéis de autoria, gêneros, demografias, editoras originais e pré-publicações. Cada autor aparece uma única vez na Obra, com um ou mais papéis. A repetição do mesmo autor é recusada com: "Autor duplicado!". Outros valores repetidos nos vínculos são recusados com "Valores duplicados nos vinculos da Obra.". Os créditos são ordenados automaticamente para exibição pelo papel de maior prioridade de cada autor, na ordem Criador Original, História Original, História e Arte, História, Arte, Ilustrador, Design de Personagens. Em caso de empate, a ordem é alfabética pelo nome do autor. Essa ordenação por crédito vale na ficha pública da Obra e na área administrativa. Nas listagens (vitrines, página do Autor e ficha da Edição), os autores seguem a posição persistida, que no cadastro é a ordem em que aparecem na lista de autores do formulário. A Web reordena essa lista pelo crédito sempre que um papel é marcado ou desmarcado. Não há ordenação manual de autores. Editoras originais e pré-publicações preservam a ordem definida pelo Administrador. As posições persistidas começam em 0, são contíguas e são normalizadas pelo sistema na ordem recebida.

#### RN0024 — Restrição de dados das Obras a domínios de referência

- **Estado:** Vigente
- **RFs dependentes:** RF0013, RF0014
- **Descrição:** os dados estruturados da Obra usam domínios controlados, sem texto livre fora dos valores permitidos.
  - **Valores nativos:** país de origem (Japão, Coreia do Sul, China, Taiwan), papéis de autoria (RN0023), demografias (Shonen, Shoujo, Seinen, Josei, Kodomo) e status da publicação original (Completa, Em andamento, Em hiato, Cancelada).
  - **Valores ativos:** autores, editoras originais e pré-publicações (listas administrativas) e tipo de Obra e gêneros (valores fixos do sistema, RN0066). Na alteração, tipos de Obra e gêneros legados ou inativos já vinculados à Obra são preservados.
  - **Compatibilidade com o país de origem:** o tipo de Obra precisa ser compatível com o país, conforme a tabela oficial: Mangá, Light Novel, Artbook e Databook para Japão; Manhwa para Coreia do Sul; Manhua para China e Taiwan; Novel para Japão, Coreia do Sul, China e Taiwan. Autores, editoras originais e pré-publicações com países relacionados precisam incluir o país da Obra. Os sem país relacionado são aceitos. A Web só oferece tipos e valores compatíveis com o país escolhido.

  As recusas dependem do tipo de valor:
  - **Referências por identificador** (tipo de Obra, autores, gêneros, editoras originais e pré-publicações) inexistentes, inativas ou incompatíveis com o país: "Um ou mais valores selecionados são inválidos."
  - **Valores nativos inválidos** (país de origem, papel de autoria, demografia ou status da publicação original fora da lista): a mensagem genérica "Preencha os campos obrigatórios da Obra.", no cadastro e na alteração.

#### RN0025 — Prevenção de duplicidade de Obras no catálogo

- **Estado:** Vigente
- **RFs dependentes:** RF0013, RF0014
- **Descrição:** não pode haver duas Obras com o mesmo título em português, desconsiderando maiúsculas, minúsculas e espaços externos. A duplicidade é recusada com: "Obra já cadastrada!". O título romanizado entra na busca pública, mas não na detecção de duplicidade. O slug é gerado a partir do título em português e recebe sufixo numérico (`-2`, `-3`...) em caso de colisão.

#### RN0026 — Política de obrigatoriedade de dados de Obras em cadastros e alterações

- **Estado:** Vigente
- **RFs dependentes:** RF0013, RF0014
- **Descrição:**
  - **API:** exige título em português, título romanizado, sinopse, país de origem, tipo de Obra, status da publicação original, capa interna e ao menos um autor com ao menos um papel. Título original, anos, editoras originais, gêneros, demografias e pré-publicações são opcionais. Os anos ficam entre 1900 e 2200. A ausência de dado obrigatório é recusada com "Preencha os campos obrigatórios da Obra.". O ano de fim anterior ao de início é recusado com "O fim da publicação original não pode ser anterior ao início.".
  - **Formulário da Web:** exige, por etapa, título em português, título romanizado, país e tipo; autores com papéis; ao menos uma editora original, status, ano de início, ano de fim (exceto “Em andamento” e “Em hiato”), ao menos um gênero, demografia (quando habilitada) e pré-publicação (exceto em lançamento direto); capa importada e sinopse. Ao avançar de etapa ou salvar, os campos ausentes são marcados como inválidos (os de texto e seleção com "Preencha o campo obrigatório.") e nada é enviado à API.
  - **Regras derivadas no formulário:** status “Em andamento” ou “Em hiato” limpa e desabilita o ano de fim. Lançamento direto limpa e desabilita demografias e pré-publicações (a API aplica a mesma limpeza). Tipos Artbook e Databook ativam e bloqueiam o lançamento direto. Demografia só se aplica ao tipo Mangá. A regra de Hentai está em RN0067.
  - **Capa:** na alteração, a capa já associada é preservada quando não há substituta. Não é possível remover a capa sem associar outra capa interna válida.
  - **Título original:** é opcional. Divergência na implementação: ver seção 6.3.

#### RN0027 — Atribuição compulsória da visibilidade privada a novas Obras

- **Estado:** Vigente
- **RFs dependentes:** RF0013
- **Descrição:** toda nova Obra nasce com visibilidade “Privado”.

#### RN0028 — Proteção da exclusão de Obras com visibilidade pública

- **Estado:** Vigente
- **RFs dependentes:** RF0015
- **Descrição:** Obra pública não pode ser excluída. A tentativa é recusada com: "Essa Obra está pública, não pode ser excluída!"

#### RN0029 — Proteção da exclusão de Obras com Edições vinculadas

- **Estado:** Vigente
- **RFs dependentes:** RF0015
- **Descrição:** a Obra só pode ser excluída depois de removidas todas as suas Edições. A tentativa é recusada com "Essa Obra possui Edições vinculadas, não pode ser excluída!". O banco também impede a exclusão do registro pai enquanto houver dependentes (restrição de chave estrangeira).

#### RN0030 — Proteção de alteração para privado em Obras com Edições públicas

- **Estado:** Vigente
- **RFs dependentes:** RF0018
- **Descrição:** a Obra não pode passar de “Público” a “Privado” enquanto houver Edição pública vinculada. A tentativa é recusada com: "Essa Obra possui Edições públicas, não pode ser rebaixada para privada!"

#### RN0066 — Tipos de Obra e gêneros controlados pelo sistema

- **Estado:** Vigente
- **RFs dependentes:** RF0013, RF0014, RF0030, RF0031, RF0032, RF0033
- **Descrição:** tipos de Obra e gêneros são valores fixos do sistema, criados pelas migrations, com identidade estável por código, e **não têm nenhuma gestão pelo Administrador**. Eles não aparecem em “Gerenciar opções” e só são usados como opções nos formulários e filtros.
  - **Tipos de Obra, na ordem oficial:** Mangá, Manhwa, Manhua, Light Novel, Novel, Artbook e Databook.
  - **Gêneros, na ordem oficial:** Aventura, Ação, Boys’ Love, Comédia, Drama, Ecchi, Esportes, Fantasia, Ficção Científica, Girls’ Love, Hentai, Mahou Shoujo, Mecha, Mistério, Música, Psicológico, Romance, Slice of Life, Sobrenatural, Suspense e Terror.

  A API recusa tentativas diretas de alterar esses valores:
  - criação: "Os valores dessa lista são controlados pelo sistema e não podem ser criados.";
  - renomeação ou alteração de países: "Esse valor é controlado pelo sistema: só é possível ativá-lo ou desativá-lo.";
  - exclusão: "Esse valor é controlado pelo sistema e não pode ser excluído."

  Divergência na implementação: ver seção 6.3. Valores legados sem correspondência oficial não são apagados. Continuam vinculados às Obras que já os usam, mas não são oferecidos em novos cadastros. Valores inativos não aparecem nos formulários nem nos filtros do catálogo público.

#### RN0067 — Conteúdo adulto obrigatório com o gênero Hentai

- **Estado:** Vigente
- **RFs dependentes:** RF0013, RF0014
- **Descrição:** a regra tem duas camadas.
  - **API:** enquanto a Obra tiver Hentai (ou gênero legado marcado como restrito a adultos) associado, a indicação de conteúdo adulto é gravada como ativada, mesmo que a requisição envie o valor desativado. Remover o Hentai não desativa a indicação. Depois da remoção, ela pode ser desativada explicitamente.
  - **Formulário da Web:** selecionar Hentai marca e bloqueia a opção de conteúdo adulto. Retirar o Hentai desmarca a opção automaticamente.

#### RN0068 — Compatibilidade entre papéis de autoria do mesmo autor

- **Estado:** Vigente
- **RFs dependentes:** RF0013, RF0014
- **Descrição:** o mesmo autor não pode receber “História e Arte” junto com “História” ou “Arte”, nem “História” e “Arte” separadamente (nesse caso usa-se “História e Arte”). No formulário, marcar um desses papéis desabilita os incompatíveis. A API recusa a combinação, no cadastro e na alteração. Divergência na implementação (mensagem da recusa): ver seção 6.3.

### 4.3. Edição e Volume

#### RN0031 — Restrição de dados de Edições a domínios de referência

- **Estado:** Vigente
- **RFs dependentes:** RF0019, RF0020
- **Descrição:** a editora brasileira é obrigatória e deve ser valor ativo da lista “Editoras brasileiras”. Acabamento, formato e miolos são opcionais. A Edição aceita até 50 miolos distintos, com a ordem de seleção preservada. Quando informados, devem ser valores ativos das listas “Tipos de capa”, “Formatos físicos” e “Miolos”. O número da edição é inteiro positivo, exibido em forma ordinal. O status de publicação no Brasil usa os valores nativos Completa, Em andamento, Em hiato e Cancelada. Referência inexistente ou inativa a editora brasileira, acabamento, formato ou miolo é recusada com: "Um ou mais valores selecionados são inválidos." Status nativo fora da lista ou número da edição que não seja inteiro positivo recebem a mensagem genérica da validação: "Preencha os campos obrigatórios da Edição." no cadastro e "Informe ao menos um campo válido para alterar." na alteração.

#### RN0032 — Prevenção de duplicidade de Edições no catálogo

- **Estado:** Vigente
- **RFs dependentes:** RF0019, RF0020
- **Descrição:** o número da edição é único dentro da Obra. A duplicidade no cadastro ou na alteração é recusada com: "Essa Obra já possui uma Edição com esse número cronológico!"

#### RN0033 — Política de obrigatoriedade de dados de Edições em cadastros e alterações

- **Estado:** Vigente
- **RFs dependentes:** RF0019, RF0020
- **Descrição:** no cadastro são obrigatórios o número da edição, a editora brasileira e o status de publicação no Brasil. A ausência é recusada com "Preencha os campos obrigatórios da Edição.". Acabamento, formato e miolo podem ser desconhecidos e permanecer não informados. A Edição não recebe capa (RN0069). Na alteração, é preciso enviar ao menos um campo válido.
  - **Contrato da API:** no cadastro, as chaves `coverTypeId`, `formatId` e `paperIds` precisam estar presentes no corpo. Acabamento e formato usam `null` para “não informado”; miolos usam `[]`. `paperIds` aceita até 50 identificadores positivos distintos e não aceita `null`. Omiti-las gera a recusa "Preencha os campos obrigatórios da Edição.". A Web sempre envia as três chaves.

#### RN0034 — Atribuição compulsória da visibilidade privada a novas Edições

- **Estado:** Vigente
- **RFs dependentes:** RF0019
- **Descrição:** toda nova Edição nasce com visibilidade “Privado”.

#### RN0035 — Proteção da exclusão de Edições com visibilidade pública

- **Estado:** Vigente
- **RFs dependentes:** RF0021
- **Descrição:** Edição pública não pode ser excluída. A tentativa é recusada com: "Essa Edição está pública, não pode ser excluída!"

#### RN0036 — Proteção da exclusão de Edições com Volumes vinculados

- **Estado:** Vigente
- **RFs dependentes:** RF0021
- **Descrição:** a Edição só pode ser excluída depois de removidos todos os seus Volumes. A tentativa é recusada com "Essa Edição possui Volumes vinculados, não pode ser excluída!". O banco também impede a exclusão enquanto houver dependentes.

#### RN0037 — Proteção da publicação de Edições vinculadas a Obras privadas

- **Estado:** Vigente
- **RFs dependentes:** RF0024
- **Descrição:** a Edição não pode ser publicada enquanto a Obra estiver privada. A tentativa é recusada com: "Essa Edição está vinculada a uma Obra privada, não pode ser publicada!"

#### RN0038 — Prevenção de duplicidade de Volumes no catálogo

- **Estado:** Vigente
- **RFs dependentes:** RF0025, RF0026
- **Descrição:** o número do Volume é único dentro da Edição. A duplicidade é recusada com: "Um volume desta edição com esse mesmo número já foi cadastrado anteriormente!"

#### RN0039 — Política de obrigatoriedade de dados de Volumes em cadastros e alterações

- **Estado:** Vigente
- **RFs dependentes:** RF0025, RF0026
- **Descrição:**
  - **Obrigatórios:** número do Volume (inteiro a partir de 0), capa interna e data de lançamento com precisão.
  - **Precisão da data:** “Completa” (padrão), “Mês e ano” ou “Ano”. O ano, entre 1900 e 2200, é sempre exigido. O mês é exigido nas precisões Completa e Mês e ano. O dia é exigido na precisão Completa e a data precisa existir no calendário.
  - **Opcionais:** indicação de volume único, número de páginas (positivo), moeda (R$, CR$, Cr$, NCz$ ou Cz$, com R$ por padrão), preço de capa (não negativo), ISBN-10 e ISBN-13 (com dígito verificador válido), link de afiliado (URL válida) e sinopse.
  - **ISBN inválido:** a API recusa o ISBN-10 ou o ISBN-13 com dígito verificador inválido. Divergência na implementação: ver seção 6.3.
  - **Alteração:** a data resultante é revalidada. Reduzir a precisão remove os componentes que deixam de existir. Data inválida é recusada com "Informe uma data de lançamento válida para o Volume.". A capa associada é preservada quando não há substituta.

  A ausência de dado obrigatório no cadastro é recusada com "Preencha os campos obrigatórios do Volume."

#### RN0040 — Atribuição compulsória da visibilidade da Edição matriz a Volumes

- **Estado:** Vigente
- **RFs dependentes:** RF0024, RF0025
- **Descrição:** o Volume reflete a visibilidade da sua Edição. No cadastro, herda o status vigente da Edição. Quando a visibilidade da Edição muda, todos os Volumes vinculados são atualizados na mesma transação. Não há alteração de visibilidade por Volume.

#### RN0041 — Proteção da exclusão de Volumes com visibilidade pública

- **Estado:** Vigente
- **RFs dependentes:** RF0027
- **Descrição:** Volume público não pode ser excluído. A mensagem especificada é "Esse Volume está público, não pode ser excluído!". Divergência na implementação: ver seção 6.3.

#### RN0042 — Proteção contra tornar privada uma Edição com Volumes vinculados a usuários

- **Estado:** Futuro (planejado)
- **RFs dependentes:** RF0024
- **Descrição:** a Edição pública não poderá passar a “Privado” quando algum Volume dela estiver em Estantes Digitais ou Listas de Desejos. A tentativa deverá ser recusada com: "Essa Edição tem Volumes vinculados a usuários, não pode ser tornada privada!" A regra depende da Estante Digital e da Lista de Desejos. O código tem apenas um ponto de extensão, sem efeito.

#### RN0069 — Capa derivada da Edição e publicação condicionada ao Volume 1

- **Estado:** Vigente
- **RFs dependentes:** RF0019, RF0022, RF0024, RF0026, RF0037, RF0039, RF0040
- **Descrição:** a Edição não tem capa própria. A capa exibida é a do Volume de número 1 da mesma Edição (no catálogo público, somente se esse Volume for público). Não se usa o Volume 0, o Volume 2 nem Volumes de outra Edição. Sem Volume 1, a Edição aparece sem capa. A publicação da Edição exige Volume 1 com capa interna e é recusada, sem alterar a hierarquia, com "Essa Edição não possui o Volume 1 com capa interna válida, não pode ser publicada!". Em Edição pública, o Volume 1 não pode ser renumerado: a tentativa é recusada com "Esse é o Volume 1 de uma Edição pública: renumerá-lo deixaria a Edição sem capa!".

### 4.4. Listas administrativas e contas

#### RN0043 — Prevenção de duplicidade de valores em listas de valores pré-cadastrados

- **Estado:** Vigente
- **RFs dependentes:** RF0030, RF0031
- **Descrição:** o texto de um valor é único dentro da categoria, desconsiderando maiúsculas, minúsculas e espaços externos. Na inclusão em lote, qualquer valor inválido ou duplicado rejeita toda a operação. As mensagens são:
  - valor já existente na lista: "Essa lista já tem esse valor cadastrado: {valor}." (ou "Essa lista já tem esses valores cadastrados: {valores}.");
  - repetição dentro do mesmo pedido: "Valores repetidos na solicitação: {valores}.";
  - renomeação para texto existente: "Essa lista já tem esse valor cadastrado!".

  Texto em branco e país ausente são recusados assim:
  - inclusão com texto em branco: a Web recusa antes de consultar a API, com "Informe o texto do novo valor." no campo. A API recusa com "Categoria e texto do valor são obrigatórios.";
  - inclusão só com separadores, como ", ,": "Informe ao menos um valor válido." (na Web e na API);
  - renomeação com texto em branco: a Web recusa com "Informe o texto do novo valor.". A API recusa com "Texto do novo valor é obrigatório.";
  - país relacionado ausente, onde exigido: "Selecione ao menos um país de origem relacionado.".

#### RN0044 — Proteção contra exclusão de valor administrativo em uso

- **Estado:** Vigente
- **RFs dependentes:** RF0032
- **Descrição:** valor vinculado a Obra ou Edição não pode ser excluído. A tentativa é recusada com: "Esse valor está vinculado a um mangá, não pode ser excluído!". Valores controlados pelo sistema nunca são excluídos (RN0066).

#### RN0045 — Política de prevenção de automodificação de privilégios

- **Estado:** Vigente
- **RFs dependentes:** RF0035
- **Descrição:** o Administrador não pode conceder nem remover o perfil Administrador da própria conta. Isso deve ser feito por outro Administrador. A tentativa é recusada com: "Você não pode alterar o nível de acesso de sua própria conta!". A Web desabilita a alteração na linha da própria conta.

### 4.5. Catálogo público

#### RN0046 — Restrição compulsória de visibilidade pública

- **Estado:** Vigente
- **RFs dependentes:** RF0036, RF0037, RF0038, RF0039, RF0040, RF0041, RF0042, RF0051
- **Descrição:** vitrines, buscas, página do Autor, listagens vinculadas e páginas de detalhe omitem qualquer Obra, Edição ou Volume privado. Um registro só aparece se toda a hierarquia acima dele for pública. Em acesso direto por URL, o sistema responde como não encontrado ("Obra não encontrada.", "Edição não encontrada." ou "Volume não encontrado."), sem indicar qual nível está restrito.

#### RN0047 — Interseção estrita de filtros de classificação

- **Estado:** Vigente
- **RFs dependentes:** RF0036, RF0037
- **Descrição:** filtros simultâneos são combinados por interseção. Nos filtros de múltipla escolha (gêneros e demografias), o resultado contém apenas Obras que atendem a todos os valores selecionados.

#### RN0048 — Controle de acesso e omissão condicional de conteúdo adulto

- **Estado:** Vigente
- **RFs dependentes:** RF0001, RF0011, RF0036, RF0037, RF0038, RF0039, RF0040, RF0041, RF0042, RF0051
- **Descrição:** a data de nascimento é dado privado e não aparece no catálogo. A idade é calculada quando necessária, sem gravar uma indicação de maioridade. A preferência de conteúdo adulto nasce desativada e só pode ser ativada com 18 anos completos, caso contrário a recusa é "Conteúdo +18 exige data de nascimento informada e 18 anos completos.". No catálogo público:
  - **Visitante:** não vê conteúdo adulto.
  - **Menor de 18 anos:** não vê conteúdo adulto, mesmo com preferência gravada.
  - **Conta sem o perfil Administrador:** vê conteúdo adulto somente se for maior de idade **e** tiver a preferência habilitada.
  - **Conta Ativada, com sessão válida e o perfil Administrador concedido:** lê conteúdo adulto independentemente de idade, preferência e perfil ativo. A exceção é só de leitura e deixa de valer quando o perfil Administrador é removido. Como a Web bloqueia as páginas públicas com o perfil Administrador ativo (RN0065), na prática a exceção vale quando essa conta navega com Usuário Padrão ativo.

  Sem autorização, a Obra é omitida tanto pela indicação adulta quanto pela associação ao gênero Hentai (ou gênero legado restrito), inclusive em acesso direto por URL, e o gênero restrito não é oferecido nos filtros. Na área administrativa, o Administrador ativo consulta todo o catálogo, sem filtro adulto. Essa exceção não permite ativar a preferência para menores.

#### RN0049 — Preservação de contexto entre vitrines

- **Estado:** Vigente
- **RFs dependentes:** RF0036, RF0037
- **Descrição:** ao alternar entre as abas de Obras e Edições, o termo de busca e a ordenação compatível são preservados, sem que a pessoa precise informá-los novamente.

#### RN0050 — Enriquecimento condicional por autenticação

- **Estado:** Futuro (planejado)
- **RFs dependentes:** RF0038, RF0039, RF0040, RF0041
- **Descrição:** respostas públicas de detalhe poderão incluir dados pessoais de posse e intenção de compra somente com sessão válida. Para visitantes, esses campos serão omitidos sem interromper a navegação. A regra depende da Estante Digital e da Lista de Desejos.

#### RN0051 — Cálculo derivado do total de Volumes da Edição

- **Estado:** Vigente
- **RFs dependentes:** RF0022, RF0037, RF0039, RF0040
- **Descrição:** o total de Volumes é sempre calculado a partir dos Volumes efetivamente vinculados, sem campo manual. Na área administrativa, conta todos os Volumes da Edição. No catálogo público, conta somente os Volumes públicos.

#### RN0070 — Navegação entre Volumes públicos da mesma Edição

- **Estado:** Vigente
- **RFs dependentes:** RF0041
- **Descrição:** o detalhe público de um Volume informa o Volume anterior e o seguinte da mesma Edição, na ordem crescente de número, considerando somente Volumes públicos. Volumes privados nunca aparecem como destino. Na Web, os botões de navegação ficam abaixo da capa e aparecem apenas quando existe destino válido. O botão do anterior fica à esquerda e o do seguinte à direita, cada um com o rótulo do Volume de destino (“Volume 2”, “Volume único”). Em Edição com um único Volume público, nenhum botão é exibido.

#### RN0071 — URLs públicas contextualizadas

- **Estado:** Vigente
- **RFs dependentes:** RF0038, RF0040, RF0041
- **Descrição:** as URLs públicas seguem a hierarquia Obra → Edição → Volume: `/obras/{slug}`, `/obras/{slug}/edicao/{editionId}` e `/obras/{slug}/edicao/{editionId}/volume/{volumeId}`. Links e retornos são gerados nesse formato. Se o slug ou a Edição da URL não corresponderem ao registro, a página responde como não encontrada ("Edição não encontrada." ou "Volume não encontrado."), assim como para identificadores não numéricos. Rotas isoladas como `/edicoes/{id}` e `/volumes/{id}` não existem e caem na página de rota não encontrada, sem redirecionamento.

#### RN0072 — Exibição de metadados editoriais ausentes

- **Estado:** Vigente
- **RFs dependentes:** RF0038, RF0039, RF0040, RF0041
- **Descrição:** metadados opcionais não informados, como título original, acabamento, formato, miolo, páginas, preço, datas, ISBN, sinopse e link de afiliado, não são preenchidos com valores inventados: o item é omitido. Os miolos são exibidos na ficha da Edição, em lista; a ficha do Volume não apresenta esse campo. Capa ausente ou que falha ao carregar é substituída pelo estado “Sem capa”. Divergência na implementação (períodos de publicação e capa da Edição): ver seção 6.3.

#### RN0052 — Resolução temporal padrão do calendário

- **Estado:** Futuro (planejado)
- **RFs dependentes:** RF0042
- **Descrição:** sem mês e ano informados, o calendário deverá usar o mês e o ano vigentes. Volumes com precisão apenas anual não deverão ser atribuídos a um mês específico.

### 4.6. Acervo pessoal (futuro)

#### RN0053 — Autenticação obrigatória para acervo pessoal

- **Estado:** Futuro (planejado)
- **RFs dependentes:** RF0043, RF0044, RF0045, RF0046, RF0047, RF0048, RF0049
- **Descrição:** as operações de Estante Digital e Lista de Desejos exigirão sessão válida. Visitantes navegam pelo catálogo, mas não criam, removem, sincronizam nem consultam acervo pessoal.

#### RN0054 — Exclusão mútua entre Estante Digital e Lista de Desejos

- **Estado:** Futuro (planejado)
- **RFs dependentes:** RF0043, RF0045
- **Descrição:** ao adicionar um Volume à Estante Digital, o sistema deverá removê-lo da Lista de Desejos do usuário, se existir.

#### RN0055 — Unicidade e estado de leitura do vínculo de posse por usuário e Volume

- **Estado:** Futuro (planejado)
- **RFs dependentes:** RF0043, RF0044, RF0045, RF0046, RF0047, RF0048, RF0049
- **Descrição:** não haverá vínculos duplicados entre o mesmo usuário e o mesmo Volume na Estante ou na Lista de Desejos. O vínculo de posse guardará o estado “lido” ou “não lido”, iniciado como “não lido” e removido junto com a posse. A Lista de Desejos não terá estado de leitura.

#### RN0056 — Restrição do acervo pessoal a registros públicos

- **Estado:** Futuro (planejado)
- **RFs dependentes:** RF0043, RF0045, RF0046, RF0047, RF0049
- **Descrição:** Volumes privados, ou de Edição ou Obra privada, não poderão ser adicionados. Vínculos existentes serão omitidos das consultas enquanto algum nível estiver privado, sem serem apagados.

#### RN0057 — Prevenção de conflito por posse ativa

- **Estado:** Futuro (planejado)
- **RFs dependentes:** RF0047
- **Descrição:** a adição à Lista de Desejos de um Volume já presente na Estante Digital deverá ser recusada, informando o conflito de posse.

### 4.7. Capas e mídia

#### RN0058 — Origem transitória da capa

- **Estado:** Vigente
- **RFs dependentes:** RF0050
- **Descrição:** a URL informada serve apenas como origem da importação. Ela é guardada somente como procedência restrita da capa interna e nunca é usada ou exibida como endereço da capa.

#### RN0059 — Autorização e segurança da importação de capa

- **Estado:** Vigente
- **RFs dependentes:** RF0050
- **Descrição:** somente o Administrador ativo importa ou descarta capas. A importação aceita apenas URL HTTPS de até 2048 caracteres, sem credenciais, cujo endereço resolva para destino público. `localhost`, redes privadas, reservadas ou locais e redirecionamentos para esses destinos são recusados, com no máximo 3 redirecionamentos. Aceita somente imagens AVIF, JPEG, PNG ou WebP, dentro dos limites configurados de tamanho (10 MB por padrão), quantidade de pixels e tempo de download (10 s por padrão). URL inválida é recusada com "Informe uma URL HTTPS pública válida para a capa." e imagem acima do tamanho, com "A imagem excede o tamanho máximo permitido.". Uma falha não cria associação nem remove a capa já associada.

#### RN0060 — Atomicidade da substituição de capa

- **Estado:** Vigente
- **RFs dependentes:** RF0014, RF0026, RF0050
- **Descrição:** a nova capa só substitui a atual depois de concluídos download, validação, processamento, armazenamento e associação. Qualquer falha preserva a capa anterior. Capas pendentes ou que ficaram sem uso são descartadas com segurança, inclusive pelo comando de limpeza de mídia. Uma capa já associada não pode ser descartada ("A capa já está associada a um registro.") nem associada a outro registro ("A capa interna informada é inválida ou já está em uso.").

#### RN0061 — Entrega interna e fallback de capas

- **Estado:** Vigente
- **RFs dependentes:** RF0016, RF0022, RF0028, RF0038, RF0039, RF0040, RF0041, RF0050
- **Descrição:** somente recursos armazenados na infraestrutura de mídia do CoMangá (Cloudflare R2) são retornados como capa. A URL pública é derivada da chave interna e da base pública configurada. Obra e Volume não podem ser cadastrados nem permanecer sem capa interna válida. A Edição usa a capa derivada (RN0069). O estado “Sem capa” da Web cobre apenas recurso ausente ou indisponível e não autoriza registro sem capa nem o uso da URL de procedência. Divergência na implementação (rótulo da capa da Edição): ver seção 6.3.

## 5. Requisitos Não Funcionais

### 5.1. Categoria: Desempenho

#### RNF01 — Desempenho de tempo de resposta do motor de busca de Obras e Edições

- **Categoria:** Desempenho
- **Prioridade:** Essencial
- **Estado:** Vigente
- **Descrição:** a busca de Obras e Edições deve retornar resultados paginados com P95 de até 500 ms sob carga normal, com conexão cliente-servidor estável de até 100 ms de latência. Atrasos causados pela rede do usuário final não contam. **Verificação:** `npm run load:test` na API, com critério “RNF01: P95 ≤ 500 ms” por estágio. O teste mede a latência ponta a ponta no cliente de teste, do envio da requisição ao recebimento completo da resposta. A medida inclui a rede entre o cliente de teste e a API e não isola o tempo de processamento no servidor. O resultado depende do ambiente e não é garantia permanente.

#### RNF02 — Capacidade de vazão e concorrência da API Pública

- **Categoria:** Desempenho
- **Prioridade:** Essencial
- **Estado:** Vigente
- **Descrição:** as rotas de consulta pública devem sustentar ao menos 20 requisições por segundo, com menos de 1% de respostas HTTP 5xx. **Verificação:** `npm run load:test`, com critério “RNF02: RPS ≥ 20 e 5xx < 1%”.

#### RNF03 — Eficiência de tráfego e limite de paginação

- **Categoria:** Desempenho
- **Prioridade:** Essencial
- **Estado:** Vigente
- **Descrição:** todas as listagens são paginadas, com no máximo 50 itens por página. Divergência na implementação: ver seção 6.3. Cada resposta deve ter no máximo 2 MB. **Verificação:** `npm run load:test` na API, com critério “RNF03: resposta ≤ 2 MB e endpoint paginado em até 50 itens”, avaliado na rota informada ao teste.

#### RNF04 — Desempenho de entrega de mídia estática (Capas)

- **Categoria:** Desempenho
- **Prioridade:** Essencial
- **Estado:** Vigente
- **Descrição:** o PostgreSQL guarda somente chaves e metadados das capas, nunca conteúdo binário ou base64. As imagens ficam no Cloudflare R2, processadas em variantes WebP 2:3 (1200×1800, 640×960 e 320×480), e são entregues por HTTPS a partir da base pública configurada. A interface usa dimensões estáveis, carregamento preguiçoso fora da área inicial e fallback para recurso ausente. Meta de projeto: TTFB P90 de até 300 ms na entrega das capas. Não há ferramenta no projeto que meça esse valor.

### 5.2. Categoria: Segurança

#### RNF05 — Armazenamento irreversível de credenciais (Hashing de Senhas)

- **Categoria:** Segurança
- **Prioridade:** Essencial
- **Estado:** Vigente
- **Descrição:** senhas nunca são armazenadas em texto puro nem com criptografia reversível. São processadas com bcrypt, com fator de custo 10 e salt por senha. Por limitação do algoritmo, a senha é limitada a 72 bytes em UTF-8 (RN0002).

#### RNF06 — Gerenciamento de sessão stateful e transporte seguro

- **Categoria:** Segurança
- **Prioridade:** Essencial
- **Estado:** Vigente
- **Descrição:** as sessões são guardadas e validadas no servidor, com revogação imediata no logout, na troca ou redefinição de senha e na exclusão da conta. O navegador recebe apenas um identificador opaco em cookie HttpOnly. O servidor guarda somente o resumo criptográfico. Em produção, o cookie é Secure. O SameSite é Strict por padrão e configurável conforme a topologia de implantação. A API aceita credenciais apenas das origens listadas na configuração de CORS e nunca transporta a sessão por localStorage ou cabeçalho Authorization.

#### RNF07 — Controle de acesso por perfil ativo via middleware

- **Categoria:** Segurança
- **Prioridade:** Essencial
- **Estado:** Vigente
- **Descrição:** o acesso às rotas administrativas é validado na camada de middleware da API, antes de qualquer regra de negócio ou transação. O middleware lê a sessão, os perfis concedidos e o perfil ativo, e só autoriza quando o perfil ativo é Administrador e continua concedido à conta. Caso contrário, responde HTTP 403 imediatamente (RN0064). A proteção de rotas da Web é complementar e não substitui a validação da API.

#### RNF17 — Expiração de sessão

- **Categoria:** Segurança
- **Prioridade:** Importante
- **Estado:** Futuro (planejado)
- **Descrição:** a sessão deverá expirar no servidor após um período definido, por tempo de vida ou por inatividade, além da revogação já existente (RN0014). Hoje não há validade por tempo nem `Max-Age` no cookie: a sessão vale até logout ou revogação. Os prazos serão definidos na implementação.

#### RNF08 — Limitação de Taxa de Autenticação (Rate Limiting)

- **Categoria:** Segurança
- **Prioridade:** Essencial
- **Estado:** Vigente
- **Descrição:** a API limita tentativas nos pontos de entrada de credenciais:
  - **Login:** após 5 falhas do mesmo IP numa janela de 5 minutos contada a partir da primeira falha, novas tentativas desse IP são bloqueadas antes de consultar o banco, com HTTP 429 e "Muitas tentativas de login. Tente novamente em alguns minutos.". Um login bem-sucedido zera a contagem. Divergência na implementação: ver seção 6.3.
  - **Solicitação e redefinição de senha por e-mail:** cada rota aceita até 5 pedidos por IP a cada 15 minutos. O excedente recebe HTTP 429, cabeçalho `Retry-After: 900` e "Muitas solicitações. Tente novamente em alguns minutos.".
  - **Alteração de senha autenticada:** após 5 falhas da mesma conta em 5 minutos, a alteração é bloqueada com "Muitas tentativas com a senha atual. Tente novamente em alguns minutos.". A contagem é por conta e não zera com a troca de IP.

  - **Cadastro e reenvio de ativação:** não têm limitação de taxa.

  Os contadores ficam em memória de cada instância da API: são zerados ao reiniciar o processo e não são compartilhados entre instâncias.

### 5.3. Categoria: Confiabilidade

#### RNF09 — Integridade Transacional e Conformidade ACID

- **Categoria:** Confiabilidade
- **Prioridade:** Essencial
- **Estado:** Vigente
- **Descrição:** escritas que envolvem várias tabelas (Obra com autores, papéis, gêneros e vínculos; propagação de visibilidade para Volumes; associação de capa; concessão e remoção de perfis) executam em transação ACID, com desfazimento total em caso de falha. Escritas concorrentes sensíveis (publicação, renumeração do Volume 1, perfis administrativos) são serializadas por bloqueios transacionais.

#### RNF10 — Integridade relacional rigorosa via constraints de banco

- **Categoria:** Confiabilidade
- **Prioridade:** Essencial
- **Estado:** Vigente
- **Descrição:** a integridade referencial é garantida por chaves estrangeiras, restrições e índices únicos. A hierarquia Obra → Edição → Volume, as referências a valores administrativos, as capas e os perfis de sistema usam exclusão restrita. A exclusão em cascata fica reservada a registros sem existência própria: sessões, perfis concedidos e tokens de redefinição da conta; os vínculos associativos da Obra (autores, papéis, gêneros, demografias, editoras originais e pré-publicações); as variantes de imagem de uma capa (MediaVariant); e as dependências de país de um valor de lista, que são apagadas com o valor dependente (o país referenciado usa exclusão restrita). Estruturas de banco mudam apenas por migrations versionadas.

#### RNF11 — Continuidade de dados e recuperação de desastres (DRP)

- **Categoria:** Confiabilidade
- **Prioridade:** Importante
- **Estado:** Vigente
- **Descrição:** o PostgreSQL está hospedado no Neon, com bancos separados para desenvolvimento, testes e implantação. As metas de continuidade são RPO de 24 horas, RTO de 4 horas e retenção mínima de 7 dias de pontos recuperáveis, por recursos do provedor ou backups lógicos equivalentes. As limitações do plano contratado devem ser registradas e, quando o plano não oferecer esses recursos, complementadas com exportações lógicas e testes periódicos de restauração. Não há, nos repositórios, procedimento versionado de backup e restauração que comprove essas metas.

#### RNF12 — Observabilidade e Tratamento de Exceções (Graceful Failure)

- **Categoria:** Confiabilidade
- **Prioridade:** Essencial
- **Estado:** Vigente
- **Descrição:** um tratador global de erros impede a queda do processo por exceções não previstas. Toda falha de requisição retorna JSON com o campo `error` em linguagem natural. As falhas que passam pelo tratador global incluem também `code`. Erros 5xx retornam apenas "Erro interno do servidor.", sem rastro de pilha nem códigos internos do ORM. Rotas inexistentes retornam "Rota não encontrada.". Exceções de servidor são registradas em log estruturado com identificador da requisição, método e rota, sem dados sensíveis nas rotas de autenticação. A API expõe verificação de saúde em `/health`.

### 5.4. Categoria: Usabilidade

#### RNF13 — Feedback semântico e comunicação de estado (Toasts)

- **Categoria:** Usabilidade
- **Prioridade:** Importante
- **Estado:** Vigente
- **Descrição:** o resultado das ações aparece em notificações flutuantes padronizadas no canto superior direito. Sucesso usa destaque verde, erro usa destaque vermelho e aviso usa destaque neutro. As notificações ficam visíveis por 5 segundos por padrão e têm botão de fechar. As mensagens são em linguagem natural, sem detalhes técnicos. Erros de campo de formulário aparecem junto ao próprio campo (padrão inline), com marcação de inválido e anúncio a leitores de tela.

#### RNF14 — Adaptabilidade de interface e navegação responsiva (Mobile-First)

- **Categoria:** Usabilidade
- **Prioridade:** Importante
- **Estado:** Vigente
- **Descrição:** o layout parte dos estilos móveis e se expande por pontos de quebra. Abaixo de 768 px, a navegação principal fica numa barra inferior fixa. De 768 px a 1023 px, fica numa barra lateral compacta, só com ícones. A partir de 1024 px, fica numa barra lateral completa, com rótulos. A página não deve ter rolagem horizontal.

#### RNF15 — Padronização geométrica e resiliência visual de capas

- **Categoria:** Usabilidade
- **Prioridade:** Importante
- **Estado:** Vigente
- **Descrição:** as capas são exibidas em proporção 2:3, com preenchimento sem distorção (corte das bordas excedentes), para evitar quebra de grade e deslocamentos de layout. Se a imagem falhar ou estiver ausente, o componente mantém a proporção e exibe o estado “Sem capa”. Divergência na implementação: ver seção 6.3. Nas telas administrativas, a Edição sem capa exibe “Sem capa (cadastre o Volume 1)”.

#### RNF16 — Design semântico de Estados Vazios (Empty States)

- **Categoria:** Usabilidade
- **Prioridade:** Importante
- **Estado:** Vigente
- **Descrição:** toda visualização que possa retornar zero registros apresenta um estado vazio explícito, com mensagem em linguagem natural e, quando houver ação útil, um botão ou link para o próximo passo (limpar filtros, voltar ao catálogo, cadastrar um registro). Estados de carregamento e de erro recuperável oferecem “Tentar novamente” quando aplicável.

## 6. Itens cancelados, fora de escopo e divergências

### 6.1. Itens cancelados

Estes itens constavam de versões anteriores e não fazem mais parte do sistema. Não devem ser reintroduzidos como campos ou funcionalidades atuais.

| Item | Onde aparecia | Situação atual |
| --- | --- | --- |
| Tipo de Edição (Tankobon, Kanzenban, 2 em 1...) | RF0019, RF0020, RF0022, RF0039, RF0040, RN0031, RN0033 | Removido do schema e da lista administrativa `tipos-edicao`, com seus valores apagados por migration. |
| Número de volumes originais da Obra | RF0013, RF0038, RN0026 | Removido por ser ambíguo entre mangás, novels, manhwas e outras formas de publicação. |
| Capa própria da Edição | RF0019, RF0022, RF0040, RN0033, RF0050 | Substituída pela capa derivada do Volume 1 (RN0069). |
| Gestão de tipos de Obra e gêneros em “Gerenciar opções” (criar, renomear, excluir, alterar países, ativar ou desativar) | RF0030, RF0031, RF0032 | Cancelada. São valores fixos do sistema (RN0066). |
| Indicador de conteúdo adulto na ficha pública da Obra | RF0038 | Não é exibido. O conteúdo adulto só controla a visibilidade (RN0048). |
| Mensagem única de privacidade de perfil ("Acesso negado: Você não tem permissão para acessar ou modificar os dados deste perfil.") | RN0022 | Não existe. As rotas da própria conta não recebem identificador de terceiros (RN0022) e as recusas administrativas seguem RN0064. |
| Nível de acesso único por conta (RF0035 antes se chamava “Alterar nível de acesso de uma conta de usuário específica”) | RF0034, RF0035, RN0005, RN0045 | Substituído pelos perfis concedidos e pelo perfil ativo (RN0062). O campo `nivel_acesso` é mantido apenas por compatibilidade. |
| Miolo único na Edição e miolo herdado no detalhe do Volume | RF0019, RF0020, RF0040, RF0041, RN0033, RN0072 | Substituído por lista ordenada de miolos na Edição; o detalhe de Volume não retorna nem exibe o campo. |
| Cloudinary como armazenamento de capas | Documentação técnica anterior | POC abandonada. O armazenamento ativo é o Cloudflare R2. |

Nenhum RF ou RN foi cancelado por inteiro. Os requisitos acima continuam vigentes sem os trechos cancelados.

### 6.2. Futuro (planejado)

- RF0042 a RF0049 e RN0042, RN0050, RN0052 a RN0057: Calendário de Lançamentos, Estante Digital e Lista de Desejos.
- RF0054: bloqueio e desbloqueio de contas.
- RNF17: expiração de sessão.

Esses requisitos descrevem o comportamento esperado para a implementação futura e não o sistema atual.

### 6.3. Divergências conhecidas entre especificação e implementação

- **RN0041:** a exclusão de Volume público é recusada com a mensagem da Obra ("Essa Obra está pública, não pode ser excluída!") em vez de "Esse Volume está público, não pode ser excluído!".
- **RN0026:** o título original é opcional, mas o formulário da Web o exige na etapa de identificação, junto com título em português, título romanizado, país e tipo. É um defeito da Web, não uma regra.
- **RNF03:** as listagens do catálogo público e as administrativas de Obras, Edições e Volumes respeitam o teto de 50 itens por página, mas as listagens administrativas de contas e de valores de listas aceitam até 100 itens por página, acima do teto de 50 especificado.
- **RN0065:** as rotas `/colecao/{slug}/edicao/{editionId}` e `/colecao/{slug}/edicao/{editionId}/selecionar/{modo}` reutilizam as páginas públicas de Edição sem a proteção de página pública. Com o perfil Administrador ativo, elas continuam acessíveis. São protótipo da Estante Digital, e a falta de proteção é um defeito a corrigir: devem seguir RN0065.
- **RN0064:** a mensagem de recusa administrativa da API está sem acentos ("voce nao possui permissao"), diferente do padrão textual do sistema.
- **Status “Bloqueada”:** é reconhecido no login, no reenvio, na recuperação e no filtro de contas, mas não há fluxo que bloqueie uma conta. O fluxo é planejado no RF0054.
- **RN0066:** a API ainda aceita ativar e desativar tipos de Obra e gêneros por requisição direta, embora esses valores não tenham gestão pelo Administrador. É uma divergência residual a remover, não uma funcionalidade. A Web guarda o código correspondente, hoje inalcançável: o carregamento com valores inativos e a ação de ativar e desativar (`toggleActive` e `includeInactive` em `useAdminOptionsPage.ts`), o aviso com cadeado e o botão “Ativar”/“Desativar” (`AdminOptionsView.tsx`) e a verificação `isSystemManagedCategory` (`adminOptionsModel.ts`). Nada disso é exibido, porque a lista de categorias da tela (`CATEGORIES`) não inclui tipos de Obra nem gêneros. É código morto a remover junto do resíduo da API.
- **RN0068:** a API recusa papéis incompatíveis para o mesmo autor com a mensagem genérica "Preencha os campos obrigatórios da Obra.", no cadastro e na alteração. A mensagem específica "Selecione apenas História e Arte, História ou Arte para o mesmo autor." existe na validação, mas não é repassada na resposta.
- **RN0039:** o ISBN com dígito verificador inválido é recusado sem as mensagens específicas "ISBN-10 inválido." e "ISBN-13 inválido.". O cadastro responde "Preencha os campos obrigatórios do Volume." e a alteração, "Informe ao menos um campo valido para alterar." (texto atual, sem acento). A Web não valida o ISBN antes de enviar.
- **RNF08 e RF0004:** quando o login é bloqueado por excesso de tentativas (HTTP 429), a Web não exibe "Muitas tentativas de login. Tente novamente em alguns minutos." e mostra apenas "Erro ao conectar com o servidor.", porque só repassa as mensagens de HTTP 401 e 403.
- **RN0072, RF0038 e RF0040:** a Web preenche períodos de publicação ausentes com textos genéricos. Na ficha da Obra, sem anos exibe “Não informado” e, só com o ano de início, “2020-??”. Na ficha da Edição, sem ano de início (sem Volumes públicos datados) exibe “Não informada” e, sem ano de fim, “2020-??”. Pela regra, o item ausente deveria ser omitido.
- **RN0072, RNF15 e RN0061:** na vitrine de Edições e no detalhe da Edição, a capa ausente aparece como “Capa indisponível”, e não com o estado especificado “Sem capa”.
- **RF0030:** a Web envia um valor de miolo com vírgula como um único valor, mas a API o separa em vários valores. Só os formatos preservam a vírgula na API.

## 7. Matriz de rastreabilidade

A coluna “Cenários” indica a seção homônima do documento `02-User-Stories/User Stories e Cenários Gherkin - CoMangá.md`.

| RF | Estado | RNs | Cenários |
| --- | --- | --- | --- |
| RF0001 | Vigente | RN0001–RN0007, RN0020, RN0048 | RF0001 |
| RF0002 | Vigente | RN0007, RN0008 | RF0002 |
| RF0003 | Vigente | RN0007, RN0009–RN0011 | RF0003 |
| RF0004 | Vigente | RN0012–RN0014, RN0062 | RF0004 |
| RF0005 | Vigente | RN0014 | RF0005 |
| RF0006 | Vigente | RN0015–RN0017 | RF0006 |
| RF0007 | Vigente | RN0002, RN0014, RN0017–RN0020 | RF0007 |
| RF0008 | Vigente | RN0002, RN0014, RN0020, RN0021, RN0073 | RF0008 |
| RF0009 | Vigente | RN0022, RN0062 | RF0009 |
| RF0010 | Vigente | RN0001, RN0004, RN0022 | RF0010 |
| RF0011 | Vigente | RN0022, RN0048 | RF0011 |
| RF0012 | Vigente | RN0014, RN0021, RN0022, RN0063 | RF0012 |
| RF0013 | Vigente | RN0023–RN0027, RN0066–RN0068 | RF0013 |
| RF0014 | Vigente | RN0023–RN0026, RN0060, RN0066–RN0068 | RF0014 |
| RF0015 | Vigente | RN0028, RN0029 | RF0015 |
| RF0016 | Vigente | RN0061 | RF0016 |
| RF0017 | Vigente | — | RF0017 |
| RF0018 | Vigente | RN0030 | RF0018 |
| RF0019 | Vigente | RN0031–RN0034, RN0069 | RF0019 |
| RF0020 | Vigente | RN0031–RN0033 | RF0020 |
| RF0021 | Vigente | RN0035, RN0036 | RF0021 |
| RF0022 | Vigente | RN0051, RN0061, RN0069 | RF0022 |
| RF0023 | Vigente | — | RF0023 |
| RF0024 | Vigente | RN0037, RN0040, RN0042 (futura), RN0069 | RF0024 |
| RF0025 | Vigente | RN0038–RN0040 | RF0025 |
| RF0026 | Vigente | RN0038, RN0039, RN0060, RN0069 | RF0026 |
| RF0027 | Vigente | RN0041 | RF0027 |
| RF0028 | Vigente | RN0061 | RF0028 |
| RF0029 | Vigente | — | RF0029 |
| RF0030 | Vigente | RN0043, RN0066 | RF0030 |
| RF0031 | Vigente | RN0043, RN0066 | RF0031 |
| RF0032 | Vigente | RN0044, RN0066 | RF0032 |
| RF0033 | Vigente | RN0066 | RF0033 |
| RF0034 | Vigente | RN0062 | RF0034 |
| RF0035 | Vigente | RN0045, RN0062, RN0063 | RF0035 |
| RF0036 | Vigente | RN0046–RN0049, RN0065 | RF0036 |
| RF0037 | Vigente | RN0046–RN0049, RN0051, RN0065, RN0069 | RF0037 |
| RF0038 | Vigente | RN0046, RN0048, RN0050 (futura), RN0061, RN0065, RN0071, RN0072 | RF0038 |
| RF0039 | Vigente | RN0046, RN0048, RN0050 (futura), RN0051, RN0061, RN0065, RN0069, RN0072 | RF0039 |
| RF0040 | Vigente | RN0046, RN0048, RN0050 (futura), RN0051, RN0061, RN0065, RN0069, RN0071, RN0072 | RF0040 |
| RF0041 | Vigente | RN0046, RN0048, RN0050 (futura), RN0061, RN0065, RN0070–RN0072 | RF0041 |
| RF0042 | Futuro | RN0046, RN0048, RN0052 | RF0042 |
| RF0043 | Futuro | RN0053–RN0056 | RF0043 |
| RF0044 | Futuro | RN0053, RN0055 | RF0044 |
| RF0045 | Futuro | RN0053–RN0056 | RF0045 |
| RF0046 | Futuro | RN0053, RN0055, RN0056 | RF0046 |
| RF0047 | Futuro | RN0053, RN0055–RN0057 | RF0047 |
| RF0048 | Futuro | RN0053, RN0055 | RF0048 |
| RF0049 | Futuro | RN0053, RN0055, RN0056 | RF0049 |
| RF0050 | Vigente | RN0058–RN0061 | RF0050 |
| RF0051 | Vigente | RN0046, RN0048, RN0065 | RF0051 |
| RF0052 | Vigente | RN0062, RN0064, RN0065 | RF0052 |
| RF0053 | Vigente | RN0014, RN0064, RN0065 | RF0053 |
| RF0054 | Futuro | RN0013 | RF0054 |

As regras transversais RN0064 (autorização administrativa) e RN0065 (separação de páginas) valem para todos os RFs dos módulos de Administração e de Catálogo Público, respectivamente, e têm cenários próprios em RF0053.
