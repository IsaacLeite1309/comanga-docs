# SERS - CoMangá

Grupo: Isaac Leite, Diogo Vieira, Klau Alves, Higor Pessoa

## 1. Introdução

Propósito: O propósito deste documento é definir e detalhar os Requisitos Funcionais, Requisitos Não Funcionais e Regras de Negócio para a primeira versão (MVP - Mínimo Produto Viável) do CoMangá. Este SERS servirá como guia para a equipe de desenvolvimento, garantindo que o produto final atenda às necessidades dos usuários e aos objetivos do projeto. O objetivo do projeto CoMangá é criar a plataforma de referência para catalogação e gerenciamento de coleções de mangás publicados no Brasil.

### Escopo

### O que será incluído (In-Scope)

- Sistema de cadastro e autenticação de usuários.

- Catálogo de mangás publicados no Brasil, com informações detalhadas.

- Funcionalidade de "Estante Digital" para o usuário gerenciar sua coleção pessoal.

- Funcionalidade de "Lista de Desejos" para o usuário gerenciar o que deseja comprar.

- Calendário de lançamentos para acompanhamento mensal dos Volumes publicados no Brasil.

- Ferramenta de busca e filtros para o catálogo.

- Painel administrativo para a curadoria do banco de dados.

### O que não será incluído (Out-of-Scope)

- Funcionalidades de rede social (ex: seguir usuários, mensagens diretas).

- Sistema de e-commerce ou venda direta de produtos.

- Aplicativo móvel nativo (a primeira versão será uma aplicação web responsiva).

- Catalogação de mangás não publicados oficialmente no Brasil.

- Sistema de avaliações e resenhas de usuários

- Informações sobre edições/volumes digitais.

### Terminologia

- SERS: Especificação de Requisitos de Software (este documento).

- Obra: Refere-se à propriedade intelectual original (a ideia do mangá), contendo os metadados universais criados no país de origem (ex: Título Original, Autor, Demografia, Gêneros).

- Edição: Refere-se ao produto físico licenciado e publicado no Brasil por uma editora local. Uma Obra pode ter múltiplas Edições (ex: Tankobon, Kanzenban, 2 em 1), cada uma com suas próprias características de formato, tipo de capa e papel.

- Volume: O objeto físico e unitário (o livro em si) que pertence a uma Edição específica e compõe a coleção do usuário.

- Estante Digital: A coleção pessoal e virtual de um usuário dentro da plataforma.

- Calendário de Lançamentos: Seção do site que lista os Volumes previstos para publicação em determinado mês e ano.

- Curadoria: Processo de adicionar, editar e manter a precisão das informações no banco de dados de mangás.

## 2. Descrição Geral

Visão do Produto: O CoMangá é uma aplicação web independente, projetada para ser a principal ferramenta de apoio para colecionadores de mangás no Brasil. Ele não substitui as lojas ou as editoras, mas atua como uma camada de organização e informação que se conecta a esse ecossistema. Sua principal dependência externa é a disponibilidade de informações públicas sobre os lançamentos das editoras. Pode integrar-se a lojas online através de programas de afiliados.

### Usuários e Contexto

- Colecionador Dedicado: Usuário com uma coleção considerável que busca eficiência, precisão e controle. Necessita de ferramentas detalhadas para marcar volumes possuídos, acompanhar o status de suas séries e gerenciar uma lista de desejos.

- Novo Entusiasta: Usuário em estágio inicial do hobby. Necessita de uma interface simples, ferramentas de descoberta, sinopses claras e um guia de lançamentos para se orientar no mercado.

- Administrador / Curador de Dados: Usuário interno com permissões elevadas. Responsável por manter a integridade, a normalização e a atualização do banco de dados (cadastrando Obras, Edições e Volumes sem duplicações). Ele garante que a plataforma funcione como uma "Única Fonte de Verdade" (Single Source of Truth) para os colecionadores, entregando a proposição de valor do CoMangá com máxima precisão técnica e histórica.

## 3. Requisitos Funcionais

Cada requisito descreve uma capacidade observável. As operações de cadastro, consulta, alteração e exclusão de uma mesma entidade são reunidas em “manter”.

| ID | Título | Descrição | Prioridade | Critério de aceitação |
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
| RF17 | Controlar visibilidade | O administrador publica ou torna privadas obras e edições. | Alta | Catálogo público passa a refletir a mudança; a visibilidade dos volumes acompanha a edição na mesma operação. |
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

## 4. Regras de Negócio:

As regras especificam condições de permissão, recusa e transformação aplicadas pelo servidor.

| ID | Título | Regra | RF relacionados |
| --- | --- | --- | --- |
| RN01 | Identidade única | Nome de usuário aceita de 3 a 20 caracteres ASCII entre letras, números e sublinhado e deve ser único; e-mail também deve ser único entre as contas, inclusive pendentes. | RF01, RF07 |
| RN02 | Senha válida | Criação e troca de senha exigem pelo menos 8 caracteres, incluindo maiúscula, minúscula, número e caractere especial, até 72 bytes em UTF-8; confirmação divergente impede a operação. | RF01, RF06 |
| RN03 | Ciclo de ativação | Conta recém-criada começa pendente e recebe o perfil padrão. O link de ativação expira em 24 horas, só pode ser usado uma vez e um reenvio para conta pendente invalida o link anterior; contas já ativadas não recebem novo link. | RF01, RF02, RF03 |
| RN04 | Acesso por sessão | Somente conta ativa com credenciais válidas recebe sessão. Encerrar sessão, alterar ou redefinir a senha e excluir a conta invalidam as sessões aplicáveis. | RF04, RF05, RF06, RF10 |
| RN05 | Recuperação sem revelação | Solicitação de recuperação não informa se o e-mail existe. Token é gerado somente para conta elegível, expira após 1 hora, é de uso único e sua substituição invalida o anterior. | RF06 |
| RN06 | Titularidade e perfis | Operações do próprio perfil usam a identidade da sessão; ações administrativas exigem perfil de administrador ativo e ainda atribuído à conta. | RF07, RF08, RF09, RF10, RF11 |
| RN07 | Atribuição administrativa | Administrador não pode alterar sua própria atribuição; o perfil padrão permanece na conta e o último administrador efetivo não pode ser removido. | RF11 |
| RN08 | Conteúdo adulto | Preferência adulta inicia desativada e só pode ser ativada após cálculo de 18 anos completos pela data de nascimento; visitantes e contas sem preferência válida não recebem obras adultas nas consultas públicas. | RF01, RF09, RF18, RF19, RF20, RF21 |
| RN09 | Domínios da obra | Tipo, autores, gêneros, editoras e revistas usam opções administrativas compatíveis com a categoria e, quando exigido, com o país; país, demografia, status e papéis usam valores controlados. Um autor aparece uma vez por obra; para ele, “História e Arte” não se combina com “História” ou “Arte”, nem estes dois papéis entre si. | RF12, RF14 |
| RN10 | Identificação da obra | O título nacional não pode duplicar o de outra obra ignorando diferenças de caixa; nova obra inicia privada. Título original é opcional no contrato atual. | RF14, RF17 |
| RN11 | Edição e capa derivada | O número cronológico identifica uma edição dentro da obra e uma nova edição inicia privada. Ela não possui capa própria: sua capa deriva do volume de número 1; publicá-la exige esse volume com capa e obra matriz pública. | RF15, RF17 |
| RN12 | Identificação do volume | O número do volume é único dentro da edição e pode ser zero. Um volume marcado como único não pode coexistir com outros volumes nessa edição enquanto a marcação permanecer. | RF16 |
| RN13 | Data e ISBN do volume | A data respeita a precisão informada (dia/mês/ano, mês/ano ou ano); ISBN-10 e ISBN-13, quando fornecidos, devem passar pela validação de dígito verificador. | RF16, RF21 |
| RN14 | Hierarquia de visibilidade | Volumes recebem a visibilidade da edição na criação e em mudanças posteriores. Obra com edição pública não pode ser tornada privada; edição não pode ser publicada se a obra estiver privada. | RF16, RF17 |
| RN15 | Exclusão protegida | Obra pública ou com edições, edição pública ou com volumes e volume público não podem ser excluídos pela administração. | RF14, RF15, RF16 |
| RN16 | Opções administrativas | Valor duplicado na categoria é recusado sem diferenciar maiúsculas; opções dependentes de país exigem compatibilidade; valor referenciado não pode ser excluído. | RF12, RF14, RF15, RF16 |
| RN17 | Catálogo público | Consulta ou URL pública só pode mostrar registros públicos em toda a cadeia obra → edição → volume e deve aplicar o filtro adulto antes de entregar dados. | RF18, RF19, RF20, RF21 |
| RN18 | Busca combinada | Filtros simultâneos restringem o resultado por interseção; trocar entre obras e edições preserva termos de busca compatíveis. | RF18 |
| RN19 | Calendário mensal | Volume com data conhecida apenas por ano não é atribuído a um mês arbitrário; sem período explícito, usa-se mês e ano correntes. | RF21 |
| RN20 | Privacidade da coleção e da lista de desejos | Só a conta autenticada acessa os próprios vínculos. Novas inclusões exigem volume público ligado a edição e obra públicas; vínculos preexistentes que se tornem privados ficam ocultos, sem exclusão automática. | RF22, RF23, RF24, RF25, RF26, RF27 |
| RN21 | Estado do volume na coleção ou na lista de desejos | Para cada conta e volume existe um único vínculo, que representa coleção (`Owned`) ou lista de desejos (`Wanted`). Incluir um volume na coleção remove-o da lista de desejos; um volume da coleção não pode ser incluído na lista de desejos. O volume entra na coleção como não lido, e seu estado de leitura só existe enquanto estiver nela. | RF22, RF23, RF24, RF25, RF26, RF27 |
| RN22 | Validação do lote da coleção | Todos os volumes incluídos ou removidos em uma alteração em lote da coleção devem pertencer à edição selecionada e estar disponíveis no catálogo da conta. A operação é atômica: erro em qualquer volume preserva os vínculos anteriores do lote inteiro. | RF23 |
| RN23 | Importação segura de capa | Apenas administrador pode importar por HTTPS; origem local/privada, conteúdo inválido ou acima dos limites configurados é recusado. A URL de origem não é usada para servir a capa. | RF13, RF14, RF16 |
| RN24 | Substituição de capa | A capa antiga permanece associada até que a nova seja validada, armazenada e vinculada com sucesso; cada obra e volume mantém sua própria capa interna. | RF13, RF14, RF16 |
| RN25 | Dados mínimos da obra | Criar obra exige título nacional, título romanizado, sinopse, tipo, país, status de publicação original, capa interna e pelo menos um autor com papel; título original, anos de publicação, gêneros, revistas e editoras originais podem estar ausentes. | RF14 |
| RN26 | Serialização e demografia | Ao marcar lançamento direto, a obra fica sem revistas de serialização e sem demografias. A seleção de gênero com código `boys-love` ou `girls-love` também limpa as demografias da obra. | RF14 |
| RN27 | Período e classificação da obra | Quando os dois anos de publicação original são informados, o fim não pode preceder o início. Obra vinculada a gênero com código `hentai` ou marcado como exclusivo para adultos é classificada como adulta mesmo que o formulário envie `adultContent=false`. | RF14, RF18 |
| RN28 | Dados mínimos da edição | Criar edição exige obra existente, editora brasileira, número cronológico positivo e status de publicação no Brasil. Acabamento e formato aceitam ausência; a lista de miolos pode estar vazia, mas não pode repetir um valor. | RF15 |
| RN29 | Dados mínimos do volume | Criar volume exige edição existente, número inteiro não negativo, capa interna e data compatível com sua precisão; páginas, preço, ISBN, link de afiliado e sinopse são opcionais. Se marcado como volume único, seu número é fixado em 1. | RF16 |
| RN30 | Total derivado de volumes | O total exibido de volumes da edição é calculado a partir dos volumes vinculados, sem cadastro manual desse número; nas consultas públicas, contam-se apenas os volumes públicos. | RF15, RF18, RF19 |
| RN31 | Atomicidade das alterações em lote | Alterações em lote da coleção, da lista de desejos e do estado de leitura são atômicas por edição: todas as inclusões, remoções ou mudanças de leitura solicitadas são confirmadas juntas; se qualquer volume falhar na validação ou gravação, nenhuma alteração do lote é persistida. | RF23, RF26, RF27 |

## 5. Requisitos Não Funcionais:

Estes critérios descrevem propriedades e limites da implementação documentada. Não se presumem metas de latência, disponibilidade ou recuperação sem medição e procedimento comprovados.

| ID | Categoria | Requisito mensurável | Verificação |
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
