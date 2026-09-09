# SERS - CoMangá

Grupo: Isaac Leite, Lucas Dalla, Klau Alves, Gabriel Mourgues, Higor Pessoa

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

### 3.1. Módulo: Autenticação e Perfil de Usuário

#### RF0001 — Cadastrar conta de acesso e disparar e-mail com link de ativação

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **RNs associadas:** RN0001, RN0002, RN0003, RN0004, RN0005, RN0006, RN0007, RN0020, RN0048
- **Descrição:** O sistema deve permitir que um usuário visitante cadastre uma nova conta de acesso mediante o recebimento obrigatório dos seguintes dados. Nome de Usuário E-mail Data de nascimento Senha Confirmação de Senha A data de nascimento deve ser válida e não pode ser futura. Imediatamente após a gravação, o sistema deve atribuir o status inicial da conta como “Pendente”, gerar um token de ativação e disparar automaticamente uma mensagem para o e-mail cadastrado contendo o link de ativação associado ao token.

#### RF0002 — Ativar conta de acesso via token de ativação

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **RNs associadas:** RN0007, RN0008
- **Descrição:** O sistema deve alterar o status de uma conta de acesso no banco de dados de "Pendente" para "Ativada" mediante o recebimento obrigatório de um token de ativação.

#### RF0003 — Reenviar e-mail com link de ativação da conta de acesso

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Importante
- **RNs associadas:** RN0009, RN0010, RN0011
- **Descrição:** O sistema deve permitir o reenvio do link de ativação para contas com status “Pendente”, mediante o recebimento do endereço de e-mail cadastrado, resultando na geração de um novo token de ativação e no disparo de uma nova mensagem.

#### RF0004 — Autenticar credenciais da conta de acesso e retornar sessão de acesso

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **RNs associadas:** RN0012, RN0013, RN0014
- **Descrição:** O sistema deve autenticar o acesso de uma conta mediante o recebimento obrigatório dos parâmetros e-mail e senha, resultando na criação de uma sessão stateful persistida no servidor. O identificador opaco da sessão deve ser enviado ao cliente por cookie HttpOnly, sem expor credenciais ou dados sensíveis.

#### RF0005 — Encerrar sessão de acesso ativa

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **RNs associadas:** N/A
- **Descrição:** O sistema deve encerrar a sessão de acesso apresentada no cookie HttpOnly da requisição, revogando o registro correspondente na persistência de sessões e removendo o cookie do cliente.

#### RF0006 — Solicitar redefinição de senha de acesso via e-mail

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **RNs associadas:** RN0015, RN0016, RN0017
- **Descrição:** O sistema deve processar a solicitação de redefinição de senha mediante o recebimento de um e-mail de acesso, resultando na geração de um token de verificação e no disparo de um e-mail vinculado a esse token.

#### RF0007 — Redefinir senha de acesso via token de redefinição

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **RNs associadas:** RN0002, RN0014, RN0017, RN0018, RN0019, RN0020
- **Descrição:** O sistema deve alterar a senha de acesso de uma conta no banco de dados mediante o recebimento obrigatório de um token de redefinição em conjunto com o recebimento obrigatório dos seguintes parâmetros: Nova Senha Confirmação de Nova Senha

#### RF0008 — Redefinir senha de acesso em sessão ativa

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Importante
- **RNs associadas:** RN0002, RN0014, RN0020, RN0021
- **Descrição:** O sistema deve alterar a senha de acesso da conta identificada pela sessão stateful ativa e validada, mediante o recebimento obrigatório dos seguintes parâmetros: Senha Atual Nova Senha Confirmação de Nova Senha Após a alteração bem-sucedida, o sistema deve revogar as sessões ativas da conta quando aplicável.

#### RF0009 — Consultar dados cadastrais da própria conta

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **RNs associadas:** RN0022
- **Descrição:** O sistema deve consultar os dados cadastrais da conta identificada pela sessão stateful ativa e validada. Ao processar a consulta, o sistema deve retornar estritamente o nome de usuário, o e-mail, o nível de acesso e a preferência de exibição de conteúdo adulto necessários à interface.

#### RF0010 — Alterar nome de usuário em sessão ativa

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Importante
- **RNs associadas:** RN0001, RN0022
- **Descrição:** O sistema deve alterar o nome de usuário da conta identificada pela sessão stateful ativa e validada, mediante o recebimento obrigatório do novo nome de usuário.

#### RF0011 — Alterar preferência de exibição de conteúdo adulto

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **RNs associadas:** RN0022, RN0048
- **Descrição:** O sistema deve alterar a preferência de exibição de conteúdo adulto da conta identificada pela sessão stateful ativa e validada, mediante o recebimento obrigatório de um valor booleano correspondente a Ativado ou Desativado. A ativação somente pode ocorrer quando o backend calcular, a partir da data de nascimento privada da conta, que o usuário possui 18 anos completos na data da solicitação.

#### RF0012 — Excluir permanentemente conta de acesso e dados vinculados

- **Módulo:** Autenticação e Perfil de Usuário
- **Prioridade:** Essencial
- **RNs associadas:** RN0021, RN0022
- **Descrição:** O sistema deve excluir permanentemente a conta identificada pela sessão stateful ativa e todos os dados vinculados que dependam exclusivamente dela, mediante o recebimento obrigatório da senha atual. A exclusão deve encerrar as sessões da conta e não deve aceitar identificador de usuário fornecido pelo cliente.

### 3.2. Módulo: Administração

#### RF0013 — Cadastrar nova Obra no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** RN0023, RN0024, RN0025, RN0026, RN0027
- **Descrição:** O sistema deve cadastrar uma nova Obra no banco de dados mediante o recebimento dos seguintes dados: Título em português Título original País de origem Tipo de Obra Capa interna importada por URL Um ou mais autores, cada qual com um ou mais papéis de autoria Status de publicação original Ano de início da publicação original Ano de fim da publicação original, quando aplicável Número de volumes originais, quando aplicável Uma ou mais editoras originais, em ordem definida Indicação de lançamento direto, sem pré-publicação Uma ou mais pré-publicações, em ordem definida, quando aplicável Uma ou mais demografias, quando aplicável Um ou mais gêneros Indicador de conteúdo adulto Os campos País de origem, Status de publicação original, Demografia e Papéis de autoria devem utilizar os valores nativos definidos pelo sistema. Autores, Tipo de Obra, Gêneros, Editoras Originais e Pré-publicações devem utilizar domínios administrativos válidos.

#### RF0014 — Alterar dados de uma Obra específica no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** RN0023, RN0024, RN0025, RN0026
- **Descrição:** O sistema deve permitir a alteração parcial ou completa dos dados de uma Obra cadastrada no banco de dados, mediante o recebimento obrigatório do seu identificador único em conjunto com o pacote dos novos dados, persistindo exclusivamente os campos que foram alterados e preservando os demais com seus valores anteriores.

#### RF0015 — Excluir uma Obra específica do banco de dados

- **Módulo:** Administração
- **Prioridade:** Importante
- **RNs associadas:** RN0028, RN0029
- **Descrição:** O sistema deve permitir a exclusão permanente de uma Obra cadastrada no banco de dados, mediante o recebimento obrigatório do seu identificador único.

#### RF0016 — Consultar coleção de Obras cadastradas no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** N/A
- **Descrição:** O sistema deve consultar, de forma paginada, os registros de Obras cadastradas no banco de dados. Ao realizar a consulta, o sistema deve retornar os seguintes metadados de resumo para cada Obra encontrada: URL derivada da capa interna Título em português e título original Autores vinculados, ordenados pela prioridade de seus papéis País de origem Tipo de Obra Quantidade de Edições vinculadas, calculada pelo sistema Status de visibilidade O sistema deve permitir busca textual parcial por título ou autor e filtros combinados por Tipo de Obra, País de origem e Visibilidade. A ordenação deve ser crescente pelo título, podendo ser invertida mediante requisição.

#### RF0017 — Consultar dados de uma Obra específica do banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** N/A
- **Descrição:** O sistema deve consultar e retornar todos os dados de um registro de Obra cadastrada no banco de dados mediante o recebimento obrigatório do seu identificador único, refletindo o estado mais recente do registro.

#### RF0018 — Alterar status de visibilidade de uma Obra específica do banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** RN0030
- **Descrição:** O sistema deve alterar o status de visibilidade de uma Obra entre “Público” e “Privado”, mediante o recebimento obrigatório de seu identificador único e do novo status, respeitando as regras hierárquicas aplicáveis às Edições vinculadas.

#### RF0019 — Cadastrar nova Edição vinculada a uma Obra no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** RN0031, RN0032, RN0033, RN0034
- **Descrição:** O sistema deve cadastrar uma nova Edição vinculada a uma Obra existente, mediante o recebimento do identificador único da Obra matriz e dos seguintes dados: Editora Brasileira Tipo de Edição Acabamento Formato Número cronológico da Edição, persistido como número inteiro Status de publicação no Brasil Capa interna da Edição importada por URL HTTPS Editora Brasileira, Tipo de Edição, Acabamento e Formato devem utilizar valores administrativos válidos. O Número cronológico e o Status de publicação no Brasil devem utilizar os valores nativos definidos pelo sistema.

#### RF0020 — Alterar dados de uma Edição específica no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** RN0031, RN0032, RN0033
- **Descrição:** O sistema deve permitir a alteração parcial ou completa dos dados de uma Edição cadastrada no banco de dados, mediante o recebimento obrigatório do seu identificador único em conjunto com o pacote dos novos dados, persistindo exclusivamente os campos que foram alterados e preservando os demais com seus valores anteriores.

#### RF0021 — Excluir uma Edição específica do banco de dados

- **Módulo:** Administração
- **Prioridade:** Importante
- **RNs associadas:** RN0035, RN0036
- **Descrição:** O sistema deve permitir a exclusão permanente de uma Edição cadastrada no banco de dados, mediante o recebimento obrigatório do seu identificador único.

#### RF0022 — Consultar coleção de Edições vinculadas a uma Obra no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** N/A
- **Descrição:** O sistema deve consultar, de forma paginada, as Edições vinculadas a uma Obra matriz. Ao realizar a consulta, o sistema deve retornar os seguintes metadados de resumo para cada Edição encontrada: URL derivada da capa interna da Edição Editora Brasileira Número cronológico da Edição, apresentado em formato ordinal Tipo de Edição Quantidade de Volumes vinculados, calculada pelo sistema Status de visibilidade A coleção deve ser ordenada de forma decrescente pelo Número cronológico da Edição.

#### RF0023 — Consultar dados de uma Edição específica do banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** N/A
- **Descrição:** O sistema deve consultar e retornar todos os dados de um registro de Edição cadastrada no banco de dados mediante o recebimento obrigatório do seu identificador único, refletindo o estado mais recente do registro.

#### RF0024 — Alterar status de visibilidade de uma Edição específica do banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** RN0037, RN0040, RN0042
- **Descrição:** O sistema deve alterar o status de visibilidade de uma Edição entre “Público” e “Privado”, mediante o recebimento obrigatório de seu identificador único e do novo status. A operação deve respeitar a visibilidade da Obra matriz e propagar o resultado aos Volumes vinculados.

#### RF0025 — Cadastrar novo Volume vinculado a uma Edição no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** RN0038, RN0039, RN0040
- **Descrição:** O sistema deve cadastrar um novo Volume vinculado a uma Edição existente, mediante o recebimento do identificador único da Edição matriz e dos seguintes dados: Número do Volume, permitindo o valor zero Indicação de Volume Único Capa interna importada por URL Precisão da data de publicação, como data completa, mês e ano ou apenas ano Data de publicação compatível com a precisão informada Moeda do preço de capa, com R$ como valor padrão Preço de capa Número de páginas ISBN-10 ISBN-13 Link afiliado Sinopse do Volume O Volume deve herdar a visibilidade vigente da Edição matriz no momento do cadastro.

#### RF0026 — Alterar dados de um Volume específico no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** RN0038, RN0039
- **Descrição:** O sistema deve permitir a alteração parcial ou completa dos dados de um Volume cadastrado no banco de dados, mediante o recebimento obrigatório de seu identificador único e do conjunto de novos dados, persistindo exclusivamente os campos alterados e preservando os demais.

#### RF0027 — Excluir um Volume específico do banco de dados

- **Módulo:** Administração
- **Prioridade:** Importante
- **RNs associadas:** RN0041
- **Descrição:** O sistema deve permitir a exclusão permanente de um Volume cadastrado no banco de dados, mediante o recebimento obrigatório do seu identificador único.

#### RF0028 — Consultar coleção de Volumes vinculados a uma Edição no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** N/A
- **Descrição:** O sistema deve consultar, de forma paginada, os Volumes vinculados a uma Edição matriz. Ao realizar a consulta, o sistema deve retornar os seguintes metadados de resumo para cada Volume encontrado: URL derivada da capa interna Número do Volume ou indicação de Volume Único Data de publicação, quando disponível Status de visibilidade A coleção deve ser ordenada de forma crescente pelo Número do Volume.

#### RF0029 — Consultar dados de um Volume específico do banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** N/A
- **Descrição:** O sistema deve consultar e retornar todos os dados de um registro de Volume cadastrado no banco de dados mediante o recebimento obrigatório do seu identificador único, refletindo o estado mais recente do registro.

#### RF0030 — Cadastrar novo valor em uma lista de valores pré-cadastrados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** RN0043
- **Descrição:** O sistema deve incluir um ou mais valores em uma categoria administrativa mediante o recebimento obrigatório da categoria alvo e do texto de cada novo valor. Quando a categoria depender de País de origem, o sistema também deve receber ao menos um país compatível. Campos nativos do formulário não devem ser expostos como listas administrativas editáveis.

#### RF0031 — Alterar um valor específico de uma lista de valores pré-cadastrados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** RN0043
- **Descrição:** O sistema deve permitir a alteração de um valor de categoria administrativa mediante o recebimento obrigatório de seu identificador único e do novo texto. Quando aplicável, também deve permitir a atualização dos países relacionados ao valor.

#### RF0032 — Excluir um valor específico de uma lista de valores pré-cadastrados

- **Módulo:** Administração
- **Prioridade:** Importante
- **RNs associadas:** RN0044
- **Descrição:** O sistema deve permitir a exclusão permanente de um valor de uma lista de valores pré-cadastrados no banco de dados, mediante o recebimento obrigatório do seu identificador único.

#### RF0033 — Consultar coleção de valores de listas pré-cadastradas

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** N/A
- **Descrição:** O sistema deve consultar, de forma paginada e pesquisável, os valores vinculados a uma categoria administrativa. O retorno deve conter o identificador, o texto do valor e, quando aplicável, os países relacionados. A consulta deve admitir ordenação alfabética crescente ou decrescente.

#### RF0034 — Consultar coleção de contas de usuário cadastradas no banco de dados

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** N/A
- **Descrição:** O sistema deve consultar, de forma paginada, as contas de usuário cadastradas no banco de dados. Para cada conta, a consulta deve retornar exclusivamente: Identificador único Nome de Usuário Endereço de E-mail Nível de Acesso atual Status de Acesso atual, entre Pendente, Ativada e Bloqueada O sistema deve permitir busca textual parcial por Nome de Usuário ou E-mail, filtros combinados por Nível de Acesso e Status e ordenação crescente ou decrescente pelo Nome de Usuário.

#### RF0035 — Alterar nível de acesso de uma conta de usuário específica

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** RN0045
- **Descrição:** O sistema deve permitir a alteração do nível de acesso de uma conta de usuário, mediante o recebimento obrigatório do seu identificador único e do novo nível de acesso.

### 3.3. Módulo: Catálogo Público e Busca

#### RF0036 — Consultar vitrine pública de Obras

- **Módulo:** Catálogo Público e Busca
- **Prioridade:** Essencial
- **RNs associadas:** RN0046, RN0047, RN0048, RN0049
- **Descrição:** O sistema deve permitir que visitantes e usuários autenticados consultem a vitrine pública de Obras mediante listagem paginada e busca textual pelos seguintes dados: Título Título Original Autor A consulta também deve admitir filtros por Tipo de Obra, País de origem, Demografia e Gênero. Os resultados devem respeitar obrigatoriamente as regras de visibilidade pública, interseção de filtros e ocultação condicional de conteúdo adulto.

#### RF0037 — Consultar vitrine pública de Edições

- **Módulo:** Catálogo Público e Busca
- **Prioridade:** Essencial
- **RNs associadas:** RN0046, RN0047, RN0048, RN0049, RN0051
- **Descrição:** O sistema deve permitir que visitantes e usuários autenticados consultem a vitrine pública de Edições físicas mediante listagem paginada e busca textual híbrida pelos seguintes dados: Título da Obra matriz Título Original da Obra matriz Autor da Obra matriz A consulta também deve admitir filtros por Editora Brasileira, Formato da Edição, Acabamento da Edição. A consulta deve retornar somente Edições públicas vinculadas a Obras públicas e deve preservar os termos fundamentais ao alternar entre as vitrines de Obras e Edições.

### 3.4. Módulo: Catálogo Público e Navegação Profunda

#### RF0038 — Consultar detalhes públicos de uma Obra

- **Módulo:** Catálogo Público e Navegação Profunda
- **Prioridade:** Essencial
- **RNs associadas:** RN0046, RN0048, RN0050
- **Descrição:** O sistema deve permitir que visitantes e usuários autenticados visualizem a ficha pública completa de uma Obra, contendo: Título e Título Original Capa País de origem e Tipo de Obra Autores e Papéis de autoria Editoras Originais e Pré-publicações Demografias e Gêneros Status, início e fim da publicação original Número de volumes originais Indicador de conteúdo adulto Edições públicas vinculadas Quando houver sessão autenticada válida, a resposta pode ser enriquecida com informações personalizadas de posse e intenção de compra.

#### RF0039 — Consultar Edições públicas vinculadas a uma Obra

- **Módulo:** Catálogo Público e Navegação Profunda
- **Prioridade:** Essencial
- **RNs associadas:** RN0046, RN0048, RN0050, RN0051
- **Descrição:** O sistema deve permitir que visitantes e usuários autenticados consultem as Edições públicas vinculadas a uma Obra pública. Para cada Edição, deve apresentar uma prévia com: Capa da Edição Número cronológico da Edição Editora Brasileira Tipo Status de publicação no Brasil Total calculado de Volumes cadastrados Amostragem dos Volumes públicos iniciais A consulta deve omitir Edições privadas e Volumes privados.

#### RF0040 — Consultar detalhes públicos de uma Edição

- **Módulo:** Catálogo Público e Navegação Profunda
- **Prioridade:** Essencial
- **RNs associadas:** RN0046, RN0048, RN0050, RN0051
- **Descrição:** O sistema deve permitir que visitantes e usuários autenticados visualizem a ficha pública completa de uma Edição, contendo: Obra matriz vinculada Capa da Edição Editora Brasileira Tipo de Edição, Acabamento e Formato Número cronológico da Edição Status de publicação no Brasil Total calculado de Volumes cadastrados Listagem paginada dos Volumes públicos vinculados A consulta deve permitir navegação contextual para a Obra matriz e para cada Volume público da Edição.

#### RF0041 — Consultar detalhes públicos de um Volume

- **Módulo:** Catálogo Público e Navegação Profunda
- **Prioridade:** Essencial
- **RNs associadas:** RN0046, RN0048, RN0050
- **Descrição:** O sistema deve permitir que visitantes e usuários autenticados visualizem a ficha pública completa de um Volume, contendo: Obra matriz e Edição vinculadas Capa e Número do Volume Indicação de Volume Único Data de publicação conforme a precisão disponível Moeda e Preço de capa Número de páginas ISBN-10 e ISBN-13 Link afiliado Sinopse do Volume A consulta deve permitir navegação contextual de retorno para a Edição e a Obra matriz.

### 3.5. Módulo: Catálogo Público e Calendário de Lançamentos

#### RF0042 — Consultar calendário público de lançamentos

- **Módulo:** Catálogo Público e Calendário de Lançamentos
- **Prioridade:** Essencial
- **RNs associadas:** RN0046, RN0048, RN0052
- **Descrição:** O sistema deve permitir que visitantes e usuários autenticados consultem os Volumes públicos previstos para determinado mês e ano, desde que a precisão da data armazenada permita identificar o mês, exibindo: Obra matriz Edição vinculada Número e capa do Volume Data de publicação Editora Brasileira A consulta deve permitir filtro por Editora Brasileira e busca textual por Título, Título Original ou Autor da Obra matriz. Na ausência de mês e ano, deve utilizar o período vigente no servidor.

### 3.6. Módulo: Estante Digital e Lista de Desejos

#### RF0043 — Registrar posse individual de Volume

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **RNs associadas:** RN0053, RN0054, RN0055, RN0056
- **Descrição:** O sistema deve permitir que um usuário autenticado registre um Volume público como adquirido, criando um vínculo entre sua conta e o Volume na Estante Digital com o estado inicial de leitura como “não lido”. Após a criação, o mesmo Volume deve ser removido automaticamente da Lista de Desejos do usuário, caso esteja presente.

#### RF0044 — Atualizar estado de leitura ou remover posse individual de Volume

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **RNs associadas:** RN0053, RN0055
- **Descrição:** O sistema deve permitir que um usuário autenticado marque como lido ou não lido um Volume que já possua na Estante Digital. O sistema também deve permitir a remoção do vínculo de posse com um Volume específico. A operação deve afetar somente o vínculo do usuário, sem excluir o Volume do catálogo; ao remover a posse, o estado de leitura associado ao vínculo deve ser removido junto.

#### RF0045 — Sincronizar registros de posse em lote

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **RNs associadas:** RN0053, RN0054, RN0055, RN0056
- **Descrição:** O sistema deve permitir que um usuário autenticado, a partir de uma Edição, adicione e/ou remova múltiplos Volumes de sua Estante Digital em uma única operação transacional. Os Volumes adicionados devem ser removidos da Lista de Desejos e devem iniciar como “não lidos”; os vínculos removidos devem eliminar junto seus estados de leitura. Os vínculos devem permanecer únicos por usuário e Volume.

#### RF0046 — Consultar registros da Estante Digital

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **RNs associadas:** RN0053, RN0055, RN0056
- **Descrição:** O sistema deve permitir que um usuário autenticado consulte os Volumes de sua Estante Digital, agrupados por Obra e Edição. A consulta deve apresentar a progressão da coleção em relação aos Volumes públicos cadastrados em cada Edição e o estado “lido” ou “não lido” de cada vínculo de posse retornado.

#### RF0047 — Registrar intenção de compra

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **RNs associadas:** RN0053, RN0055, RN0056, RN0057
- **Descrição:** O sistema deve permitir que um usuário autenticado adicione um Volume público à sua Lista de Desejos. Antes de criar o vínculo, deve verificar se o usuário já possui o mesmo Volume na Estante Digital.

#### RF0048 — Remover intenção de compra

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **RNs associadas:** RN0053, RN0055
- **Descrição:** O sistema deve permitir que um usuário autenticado remova um Volume de sua Lista de Desejos. A operação deve afetar somente o vínculo do usuário, sem excluir o Volume do catálogo.

#### RF0049 — Consultar Lista de Desejos

- **Módulo:** Estante Digital e Lista de Desejos
- **Prioridade:** Essencial
- **RNs associadas:** RN0053, RN0055, RN0056
- **Descrição:** O sistema deve permitir que um usuário autenticado consulte os Volumes de sua Lista de Desejos com seus dados públicos essenciais. Registros que deixarem de estar públicos em qualquer nível da hierarquia devem ser omitidos enquanto permanecerem indisponíveis.

#### RF0050 — Importar e gerenciar capa interna por URL

- **Módulo:** Administração
- **Prioridade:** Essencial
- **RNs associadas:** RN0058, RN0059, RN0060, RN0061
- **Descrição:** O sistema deve permitir que um Administrador importe individualmente uma capa para uma Obra, Edição ou Volume mediante uma URL HTTPS. O backend deve baixar, validar, processar e armazenar a imagem na infraestrutura de mídia controlada pelo CoMangá antes de associá-la à entidade. A operação deve permitir importação e substituição atômica, sem utilizar a URL externa como referência de exibição da capa. Não deve ser permitida a remoção de uma capa associada sem a substituição bem-sucedida por outra capa interna válida.

## 4. Regras de Negócio:

#### RN0001 — Política de formatação de Username da conta de acesso

- **RFs dependentes:** RF0001, RF0010
- **Descrição:** Para o preenchimento ou alteração do Nome de Usuário, o sistema deve exigir o cumprimento simultâneo das seguintes condições de formatação: Ter comprimento entre 3 e 20 caracteres. Conter exclusivamente caracteres alfanuméricos (letras sem acentos e números) e o caractere sublinhado (_). Não conter espaços em branco ou quaisquer outros caracteres especiais. Caso o usuário tente submeter um Nome de Usuário que viole qualquer uma destas condições, o sistema deve recusar a transação e retornar a seguinte mensagem de erro específica para o campo: "Utilize entre 3 e 20 caracteres, sem espaços, acentos ou caracteres especiais."

#### RN0002 — Política de força da senha da conta de acesso

- **RFs dependentes:** RF0001, RF0006, RF0007, RF0008, RF0012
- **Descrição:** Para a definição ou alteração da senha de acesso, o sistema deve exigir simultaneamente: Comprimento mínimo de 8 caracteres. Ao menos uma letra maiúscula. Ao menos uma letra minúscula. Ao menos um número. Ao menos um caractere especial. Caso a senha viole qualquer condição, o sistema deve recusar a transação e retornar: "Utilize no mínimo 8 caracteres, incluindo pelo menos uma letra maiúscula, uma minúscula, um número e um caractere especial."

#### RN0003 — Prevenção de duplicidade de e-mails de contas de acesso

- **RFs dependentes:** RF0001
- **Descrição:** O sistema deve garantir a unicidade absoluta de endereços de e-mail das contas de acesso no banco de dados, impedindo a colisão de credenciais. Durante qualquer tentativa de cadastro de uma nova conta, o sistema deve exigir que o e-mail submetido não esteja vinculado a nenhuma outra conta existente, independentemente do status atual dessa conta (Pendente, Ativada). Caso essa condição seja violada, o sistema deve recusar a transação e retornar o erro: "Este endereço de e-mail já está em uso. Tente fazer login ou recuperar sua senha."

#### RN0004 — Prevenção de duplicidade de nomes de usuário

- **RFs dependentes:** RF0001, RF0010
- **Descrição:** O sistema deve garantir a unicidade absoluta dos nomes de usuário no banco de dados. Durante cadastros ou alterações, o nome submetido não pode estar vinculado a outra conta. Caso essa condição seja violada, o sistema deve recusar a transação e retornar: "Este nome de usuário não está disponível. Por favor, escolha outro."

#### RN0005 — Atribuição compulsória de nível de acesso padrão a novas contas

- **RFs dependentes:** RF0001
- **Descrição:** Toda nova conta deve receber compulsoriamente o nível de acesso “Usuário Padrão”.

#### RN0006 — Atribuição compulsória da preferência não exibição de conteúdo adulto a novas contas

- **RFs dependentes:** RF0001
- **Descrição:** Para toda nova conta de acesso, o sistema deve atribuir compulsoriamente a preferência de exibição de conteúdo +18 como "Desativada".

#### RN0007 — Tempo de expiração do token de ativação da conta de acesso

- **RFs dependentes:** RF0001, RF0002, RF0003
- **Descrição:** O tempo de expiração do token de ativação é de 24 horas. Se o usuário tentar verificar a conta com um token expirado, o sistema deve recusar a transação, e retornar a mensagem de erro: "Este link de ativação expirou. Solicite um novo e-mail de ativação."

#### RN0008 — Invalidação por unicidade do token de ativação

- **RFs dependentes:** RF0002
- **Descrição:** O token de ativação da conta de acesso se torna inválido imediatamente após o primeiro uso bem-sucedido. Se o usuário tentar ativar a conta com um token de ativação invalidado, o sistema deve recusar a transação e retornar uma mensagem de erro “Link de ativação inválido!”.

#### RN0009 — Recusa do reenvio do link de ativação a contas não existentes

- **RFs dependentes:** RF0003
- **Descrição:** Caso o e-mail informado para reenvio do link de ativação não exista no banco de dados, o sistema deve recusar a transação e retornar: "Endereço de e-mail não cadastrado."

#### RN0010 — Recusa do reenvio do link de ativação a contas ativadas

- **RFs dependentes:** RF0003
- **Descrição:** Caso o e-mail fornecido ao pedir o reenvio do link de ativação pertença a uma conta com o status de acesso "Ativada", o sistema deve recusar a transação e retornar uma mensagem erro: “Este endereço de e-mail pertence a uma conta ativada.”

#### RN0011 — Invalidação por sobreposição do novo token de ativação

- **RFs dependentes:** RF0003
- **Descrição:** A geração de um novo token de ativação deve invalidar imediatamente qualquer token de ativação anterior vinculado àquela conta. Se o usuário tentar ativar a conta com um token de ativação invalidado, o sistema deve recusar a transação e retornar uma mensagem de erro “Link de ativação inválido!”.

#### RN0012 — Bloqueio de sessão por credenciais inválidas

- **RFs dependentes:** RF0004
- **Descrição:** Caso o usuário tente autenticar-se com e-mail e/ou senha que não correspondam a uma conta válida, o sistema deve recusar a transação e retornar: "Credenciais inválidas!"

#### RN0013 — Bloqueio de sessão por status de acesso pendente

- **RFs dependentes:** RF0004
- **Descrição:** Caso o usuário realize uma tentativa de login em uma conta que está com o status de acesso “Pendente”, o sistema deve recusar a transação e retornar a mensagem de erro: “Conta de acesso pendente. Ative a conta com o e-mail de verificação enviado anteriormente.”

#### RN0014 — Política de persistência de sessão de acesso

- **RFs dependentes:** RF0004, RF0005, RF0007, RF0008
- **Descrição:** A autenticação deve utilizar sessão stateful persistida no servidor. A cada login bem-sucedido, o sistema deve gerar um identificador opaco e criptograficamente seguro, armazenar somente seu hash na tabela de sessões e enviar o valor original ao cliente por cookie HttpOnly. A sessão deve ser considerada válida apenas enquanto existir, não estiver revogada e estiver vinculada a uma conta apta ao acesso. O ciclo de vida da sessão deve ser encerrado, no mínimo, nas seguintes situações: Logout explícito do usuário. Redefinição ou alteração de senha, quando aplicável. Exclusão da conta. Caso uma rota protegida receba uma sessão ausente, inválida ou revogada, o sistema deve retornar: "Sua sessão é inválida ou foi encerrada. Por favor, faça login novamente."

#### RN0015 — Recusa da solicitação de redefinição de senha a contas inexistentes

- **RFs dependentes:** RF0006
- **Descrição:** Caso o usuário realize uma solicitação de redefinição de senha com um e-mail que não coincide com nenhum cadastro previamente cadastrado no banco de dados, o sistema deve recusar a transação e retornar uma mensagem de erro: “E-mail não cadastrado!”.

#### RN0016 — Recusa da solicitação de redefinição de senha a contas com status de acesso pendente

- **RFs dependentes:** RF0006
- **Descrição:** Caso o usuário realize uma solicitação de redefinição de senha em uma conta que está com o status de acesso “Pendente”, o sistema deve recusar a transação e retornar a mensagem de erro: “Ative a conta com o e-mail de verificação enviado anteriormente para alterar a senha.”

#### RN0017 — Tempo de expiração do token de redefinição de senha de acesso

- **RFs dependentes:** RF0006, RF0007
- **Descrição:** O tempo de expiração do token de redefinição é de 1 hora. Se o usuário tentar redefinir a senha com um token expirado, o sistema deve recusar a transação, e retornar a mensagem de erro: "Este link de redefinição expirou. Solicite a redefinição novamente."

#### RN0018 — Invalidação por unicidade do token de redefinição

- **RFs dependentes:** RF0007
- **Descrição:** O token de redefinição de senha torna-se inválido imediatamente após o primeiro uso bem-sucedido. Uma nova tentativa com o mesmo token deve ser recusada com a mensagem: "Link de redefinição inválido!"

#### RN0019 — Invalidação por sobreposição do novo token de redefinição

- **RFs dependentes:** RF0007
- **Descrição:** A geração de um novo token de redefinição deve invalidar imediatamente qualquer token de redefinição anterior vinculado àquela conta. Se o usuário tentar redefinir a senha com um token de redefinição invalidado, o sistema deve recusar a transação e retornar uma mensagem de erro “Link de redefinição inválido!”.

#### RN0020 — Recusa de transação por divergência em confirmação de senha

- **RFs dependentes:** RF0001, RF0007, RF0008
- **Descrição:** Caso o usuário tente realizar uma transação que exija a criação ou alteração de credenciais e os parâmetros de "Senha" e "Confirmação de Senha" apresentem valores diferentes, o sistema deve recusar a requisição e retornar a mensagem de erro: "Divergência nos valores da senha e confirmação de senha!"

#### RN0021 — Recusa de transação por divergência de credencial vigente

- **RFs dependentes:** RF0008, RF0012
- **Descrição:** Caso o usuário tente realizar uma transação de alto privilégio em sessão ativa que exija validação de segurança adicional, e o parâmetro de "Senha Atual" enviado não corresponda à senha vigente da conta de acesso registrada no banco de dados, o sistema deve recusar a transação e retornar a mensagem de erro: "Senha atual incorreta!"

#### RN0022 — Política de isolamento e privacidade de manipulação de dados do perfil

- **RFs dependentes:** RF0009, RF0010, RF0011, RF0012
- **Descrição:** A consulta, alteração e exclusão dos dados do perfil devem utilizar exclusivamente a identidade obtida da sessão stateful validada. Rotas destinadas ao próprio usuário não devem aceitar o identificador da conta pelo corpo ou pela URL. Em rotas administrativas, o sistema deve validar explicitamente o privilégio necessário. Qualquer tentativa não autorizada deve ser recusada com: "Acesso negado: Você não tem permissão para acessar ou modificar os dados deste perfil."

#### RN0023 — Política de multiplicidade de vínculos da Obra

- **RFs dependentes:** RF0013, RF0014
- **Descrição:** Uma Obra pode possuir múltiplos autores, papéis de autoria, gêneros, demografias, editoras originais e pré-publicações. Cada autor deve aparecer uma única vez na Obra, podendo receber um ou mais papéis. Autores devem ser ordenados automaticamente pela maior prioridade entre seus papéis; editoras originais e pré-publicações devem preservar a ordem definida pelo administrador. A repetição do mesmo autor na requisição deve ser recusada com: "Autor duplicado!"

#### RN0024 — Restrição de dados das Obras a domínios de referência

- **RFs dependentes:** RF0013, RF0014
- **Descrição:** Os dados estruturados das Obras devem utilizar domínios controlados, sendo vedado o uso de texto livre fora dos valores permitidos. Os seguintes campos devem utilizar valores nativos definidos pelo sistema: País de origem Papéis de autoria Demografias Status de publicação original Os seguintes campos devem utilizar valores administrativos cadastrados e ativos: Autores Tipo de Obra Gêneros Editoras Originais Pré-publicações Autores, Tipos de Obra, Editoras Originais e Pré-publicações devem respeitar as relações de compatibilidade com o País de origem quando estas estiverem definidas.

#### RN0025 — Prevenção de duplicidade de Obras no catálogo

- **RFs dependentes:** RF0013, RF0014
- **Descrição:** Durante o cadastro ou a alteração de uma Obra, o sistema deve impedir a existência de outra Obra com o mesmo título em português, desconsiderando diferenças entre letras maiúsculas e minúsculas e espaços externos. Caso encontre duplicidade, deve recusar a transação e retornar: "Uma Obra com esse mesmo título já foi cadastrada anteriormente!"

#### RN0026 — Política de obrigatoriedade de dados de Obras em cadastros e alterações

- **RFs dependentes:** RF0013, RF0014
- **Descrição:** No cadastro de Obras, são obrigatórios quando aplicáveis: Título em português Título original País de origem Tipo de Obra Capa interna importada por URL Ao menos um autor com ao menos um papel Status de publicação original Ano de início da publicação original Ao menos uma Editora Original Ao menos um Gênero Indicador de conteúdo adulto Em alteração parcial, a capa interna já associada deve ser preservada quando não houver substituta. Não é permitida a remoção da capa sem a associação bem-sucedida de outra capa interna válida. O Ano de fim e o Número de volumes originais deixam de ser exigidos quando o status for “Em andamento” ou “Em hiato”. Pré-publicação deixa de ser exigida quando a Obra for marcada como lançamento direto. Demografia deixa de ser exigida quando o Tipo de Obra for incompatível ou quando houver lançamento direto. A seleção do gênero Hentai deve ativar e bloquear o indicador de conteúdo adulto. Artbook e Databook devem ativar e bloquear a opção de lançamento direto. A ausência de qualquer dado obrigatório aplicável deve resultar em recusa da transação com: "Dados obrigatórios faltando!"

#### RN0027 — Atribuição compulsória da visibilidade privada a novas Obras

- **RFs dependentes:** RF0013
- **Descrição:** Para toda nova Obra registrada, o sistema deve atribuir compulsoriamente o status de visibilidade “Privado”.

#### RN0028 — Proteção da exclusão de Obras com visibilidade pública

- **RFs dependentes:** RF0015
- **Descrição:** O sistema não deve permitir a exclusão de Obras com visibilidade “Público”. A tentativa deve ser recusada com: "Essa Obra está pública, não pode ser excluída!"

#### RN0029 — Proteção da exclusão de Obras com Edições vinculadas

- **RFs dependentes:** RF0015
- **Descrição:** Uma Obra somente pode ser excluída depois que todas as Edições vinculadas tiverem sido removidas. A relação entre Obra e Edição deve utilizar ON DELETE RESTRICT, impedindo fisicamente a exclusão do registro pai enquanto existirem dependências e evitando registros órfãos.

#### RN0030 — Proteção de alteração para privado em Obras com Edições públicas

- **RFs dependentes:** RF0018
- **Descrição:** O sistema deve bloquear a alteração de uma Obra de “Público” para “Privado” enquanto existir ao menos uma Edição pública vinculada. A tentativa deve ser recusada com: "Essa Obra possui Edições públicas, não pode ser rebaixada para privada!"

#### RN0031 — Restrição de dados de Edições a domínios de referência

- **RFs dependentes:** RF0019, RF0020
- **Descrição:** Os dados estruturados de Edições devem utilizar domínios controlados. Os seguintes campos devem utilizar valores administrativos cadastrados e ativos: Editora Brasileira Tipo de Edição Acabamento Formato O Número cronológico da Edição deve ser persistido como número inteiro positivo e apresentado em formato ordinal na interface. O Status de publicação no Brasil deve utilizar os valores nativos definidos pelo sistema.

#### RN0032 — Prevenção de duplicidade de Edições no catálogo

- **RFs dependentes:** RF0019, RF0020
- **Descrição:** O sistema deve, durante qualquer tentativa de cadastro ou alteração de Edições, validar se a Edição não fere unicidade do catálogo, utilizando uma chave composta de Identificador Único da Obra matriz + Número Cronológico da Edição. Caso o sistema identifique uma Edição no banco de dados que tem esses mesmos dados idênticos, deve recusar a transação e retornar o erro: "Uma Edição com esse mesmo número cronológico já foi cadastrada anteriormente!"

#### RN0033 — Política de obrigatoriedade de dados de Edições em cadastros e alterações

- **RFs dependentes:** RF0019, RF0020
- **Descrição:** Os seguintes dados são obrigatórios ao cadastrar uma Edição: Editora Brasileira Tipo de Edição Acabamento Formato Número cronológico da Edição Status de publicação no Brasil Capa interna importada por URL Em alteração parcial, a capa interna já associada deve ser preservada quando não houver substituta. Não é permitida a remoção da capa sem a associação bem-sucedida de outra capa interna válida. A ausência de qualquer dado obrigatório deve resultar em recusa da transação com: "Dados obrigatórios faltando!"

#### RN0034 — Atribuição compulsória da visibilidade privada a novas Edições

- **RFs dependentes:** RF0019
- **Descrição:** Para toda nova Edição registrada, o sistema deve atribuir compulsoriamente o status de visibilidade “Privado”.

#### RN0035 — Proteção da exclusão de Edições com visibilidade pública

- **RFs dependentes:** RF0021
- **Descrição:** O sistema não deve permitir a exclusão de Edições com visibilidade “Público”. A tentativa deve ser recusada com: "Essa Edição está pública, não pode ser excluída!"

#### RN0036 — Proteção da exclusão de Edições com Volumes vinculados

- **RFs dependentes:** RF0021
- **Descrição:** Uma Edição somente pode ser excluída depois que todos os Volumes vinculados tiverem sido removidos. A relação entre Edição e Volume deve utilizar ON DELETE RESTRICT, impedindo fisicamente a exclusão do registro pai enquanto existirem dependências e evitando registros órfãos.

#### RN0037 — Proteção da publicação de Edições vinculadas a Obras privadas

- **RFs dependentes:** RF0024
- **Descrição:** O sistema deve bloquear a alteração de uma Edição de “Privado” para “Público” quando sua Obra matriz estiver privada. A tentativa deve ser recusada com: "Essa Edição está vinculada a uma Obra privada, não pode ser publicada!"

#### RN0038 — Prevenção de duplicidade de Volumes no catálogo

- **RFs dependentes:** RF0025, RF0026
- **Descrição:** O sistema deve, durante qualquer tentativa de cadastro ou alteração de Volumes, validar se o Volume não fere unicidade do catálogo, utilizando uma chave composta de Identificador Único da Edição matriz + Número do Volume. Caso o sistema identifique um Volume no banco de dados que tem esses mesmos dados idênticos, deve recusar a transação e retornar o erro: "Um volume desta edição com esse mesmo número já foi cadastrado anteriormente!"

#### RN0039 — Política de obrigatoriedade de dados de Volumes em cadastros e alterações

- **RFs dependentes:** RF0025, RF0026
- **Descrição:** Os seguintes dados são obrigatórios ao cadastrar um Volume: Número do Volume, aceitando valores inteiros a partir de zero Capa interna importada por URL Precisão e valor da data de publicação Os seguintes dados são opcionais: Indicação de Volume Único Número de páginas Moeda e preço de capa ISBN-10 ISBN-13 Link afiliado Sinopse do Volume Em alteração parcial, a capa interna já associada deve ser preservada quando não houver substituta. Não é permitida a remoção da capa sem a associação bem-sucedida de outra capa interna válida. A data deve aceitar precisão completa, mês e ano ou apenas ano. A ausência de qualquer dado obrigatório deve resultar em recusa da transação com: "Dados obrigatórios faltando!"

#### RN0040 — Atribuição compulsória da visibilidade da Edição matriz a Volumes

- **RFs dependentes:** RF0024, RF0025
- **Descrição:** Todo Volume deve refletir a visibilidade de sua Edição matriz. No cadastro, o Volume deve herdar o status vigente da Edição; quando a visibilidade da Edição for alterada, todos os Volumes vinculados devem ser atualizados na mesma transação.

#### RN0041 — Proteção da exclusão de Volumes com visibilidade pública

- **RFs dependentes:** RF0027
- **Descrição:** O sistema não deve permitir a exclusão de Volumes com visibilidade “Público”. A tentativa deve ser recusada com: "Esse Volume está público, não pode ser excluído!"

#### RN0042 — Proteção contra tornar privada uma Edição com Volumes vinculados a usuários

- **RFs dependentes:** RF0024
- **Descrição:** O sistema não deve permitir que uma Edição pública seja alterada para “Privado” quando qualquer Volume vinculado possuir registros em Estantes Digitais ou Listas de Desejos. A tentativa deve ser recusada com: "Essa Edição tem Volumes vinculados a usuários, não pode ser tornada privada!"

#### RN0043 — Prevenção de duplicidade de valores em listas de valores pré-cadastrados

- **RFs dependentes:** RF0030, RF0031
- **Descrição:** Durante o cadastro ou a alteração de valores administrativos, o sistema deve garantir a unicidade do texto dentro da mesma categoria, desconsiderando diferenças entre maiúsculas e minúsculas e espaços externos. Em operações em lote, a existência de qualquer valor inválido ou duplicado deve rejeitar toda a operação. A tentativa deve retornar: "Essa lista já tem esse valor cadastrado!"

#### RN0044 — Proteção contra exclusão de valor administrativo em uso

- **RFs dependentes:** RF0032
- **Descrição:** O sistema não deve permitir a exclusão de um valor administrativo enquanto ele estiver vinculado a uma Obra, Edição ou Volume. A tentativa deve ser recusada com: "Esse valor está vinculado a um mangá, não pode ser excluído!"

#### RN0045 — Política de prevenção de automodificação de privilégios

- **RFs dependentes:** RF0035
- **Descrição:** Um Administrador não pode alterar o nível de acesso da própria conta. A promoção ou o rebaixamento dessa conta deve ser realizado por outro Administrador. A tentativa deve ser recusada com: "Você não pode alterar o nível de acesso de sua própria conta!"

#### RN0046 — Restrição compulsória de visibilidade pública

- **RFs dependentes:** RF0036, RF0037, RF0038, RF0039, RF0040, RF0041, RF0042
- **Descrição:** Vitrines, buscas, calendários, listagens vinculadas e páginas públicas de detalhe devem omitir qualquer Obra, Edição ou Volume com visibilidade “Privado”. A regra também deve valer para acesso direto por identificador ou URL, sem expor os dados do registro privado.

#### RN0047 — Interseção estrita de filtros de classificação

- **RFs dependentes:** RF0036, RF0037
- **Descrição:** Quando múltiplos filtros forem aplicados simultaneamente, o sistema deve utilizar interseção lógica. Se houver múltiplos valores dentro de uma classificação combinável, o resultado deve conter apenas registros que atendam a todos os critérios informados.

#### RN0048 — Controle de acesso e omissão condicional de conteúdo adulto

- **RFs dependentes:** RF0001, RF0011, RF0036, RF0037, RF0038, RF0039, RF0040, RF0041, RF0042
- **Descrição:** A data de nascimento é dado privado da conta e não deve ser exposta no catálogo público. Toda nova conta deve iniciar com a preferência de exibição de conteúdo adulto desativada. O backend somente pode ativar essa preferência quando a data de nascimento for válida, não futura e o cálculo na data da solicitação indicar 18 anos completos. Visitantes, usuários com a preferência desativada e usuários que não atendam à maioridade devem receber conteúdo adulto omitido das áreas públicas. A área administrativa não deve aplicar essa ocultação. O sistema não deve persistir uma flag independente de maioridade; a idade deve ser calculada quando necessária.

#### RN0049 — Preservação de contexto entre vitrines

- **RFs dependentes:** RF0036, RF0037
- **Descrição:** Ao alternar entre as vitrines de Obras e Edições, o sistema deve preservar os termos fundamentais de busca compatíveis, especialmente Título e Autor, evitando que o usuário precise informá-los novamente.

#### RN0050 — Enriquecimento condicional por autenticação

- **RFs dependentes:** RF0038, RF0039, RF0040, RF0041
- **Descrição:** Respostas públicas de detalhes podem incluir dados personalizados de posse e intenção de compra somente quando houver sessão autenticada válida. Para visitantes anônimos, esses campos devem ser omitidos sem interromper a navegação nem produzir erro de autenticação.

#### RN0051 — Cálculo derivado do total de Volumes da Edição

- **RFs dependentes:** RF0037, RF0039, RF0040
- **Descrição:** O total de Volumes de uma Edição deve ser calculado a partir da quantidade de Volumes efetivamente vinculados no banco de dados. O sistema não deve manter um total manual independente que possa divergir da relação real.

#### RN0052 — Resolução temporal padrão do calendário

- **RFs dependentes:** RF0042
- **Descrição:** Quando a consulta ao calendário não informar mês e ano, o sistema deve utilizar o mês e o ano vigentes no servidor. Volumes cuja data possua precisão apenas anual não devem ser atribuídos arbitrariamente a um mês específico.

#### RN0053 — Autenticação obrigatória para acervo pessoal

- **RFs dependentes:** RF0043, RF0044, RF0045, RF0046, RF0047, RF0048, RF0049
- **Descrição:** Operações de Estante Digital e Lista de Desejos exigem sessão stateful válida. Visitantes podem navegar pelo catálogo público, mas não podem criar, remover, sincronizar ou consultar vínculos de acervo pessoal.

#### RN0054 — Exclusão mútua entre Estante Digital e Lista de Desejos

- **RFs dependentes:** RF0043, RF0045
- **Descrição:** Quando um Volume for adicionado com sucesso à Estante Digital, o sistema deve remover esse mesmo Volume da Lista de Desejos do usuário, caso exista, impedindo a coexistência de posse e intenção de compra.

#### RN0055 — Unicidade e estado de leitura do vínculo de posse por usuário e Volume

- **RFs dependentes:** RF0043, RF0044, RF0045, RF0046, RF0047, RF0048, RF0049
- **Descrição:** Um usuário não pode possuir vínculos duplicados com o mesmo Volume na Estante Digital nem na Lista de Desejos. O vínculo de posse na Estante deve armazenar o estado de leitura “lido” ou “não lido”, iniciado como “não lido”, e esse estado só pode ser alterado enquanto o vínculo existir. Ao remover a posse, o estado de leitura deve ser removido junto. A Lista de Desejos não possui estado de leitura. A camada de persistência deve garantir a unicidade por usuário e Volume em cada relação.

#### RN0056 — Restrição do acervo pessoal a registros públicos

- **RFs dependentes:** RF0043, RF0045, RF0046, RF0047, RF0049
- **Descrição:** Volumes privados ou vinculados a Edições ou Obras privadas não podem ser adicionados à Estante Digital nem à Lista de Desejos. Vínculos já existentes devem ser omitidos das consultas pessoais enquanto qualquer nível da hierarquia estiver privado, sem serem apagados automaticamente.

#### RN0057 — Prevenção de conflito por posse ativa

- **RFs dependentes:** RF0047
- **Descrição:** O sistema deve recusar a adição de um Volume à Lista de Desejos quando o usuário já possuir esse Volume na Estante Digital. A operação deve ser abortada com HTTP 409 e mensagem informando o conflito de posse ativa.

#### RN0058 — Origem transitória da capa

- **RFs dependentes:** RF0050
- **Descrição:** A URL informada pelo Administrador deve servir somente como origem da importação. Ela não pode ser persistida nem apresentada como referência de exibição da capa. Sua retenção em campo separado é permitida apenas como dado restrito de procedência e auditoria.

#### RN0059 — Autorização e segurança da importação de capa

- **RFs dependentes:** RF0050
- **Descrição:** Somente Administradores autenticados podem importar ou substituir capas. A importação deve aceitar somente HTTPS e rejeitar destinos privados, reservados ou locais, redirecionamentos proibidos, conteúdo que não seja imagem e arquivos que excedam os limites de tempo, tamanho, dimensões ou quantidade de pixels definidos pelo sistema. Uma falha de importação não pode criar uma associação incompleta nem remover a capa interna previamente associada.

#### RN0060 — Atomicidade da substituição de capa

- **RFs dependentes:** RF0050
- **Descrição:** Uma nova capa somente pode substituir a atual depois que download, validação, processamento, armazenamento e associação forem concluídos. Qualquer falha deve preservar a capa anterior e permitir a limpeza segura de objetos temporários ou órfãos.

#### RN0061 — Entrega interna e fallback de capas

- **RFs dependentes:** RF0016, RF0022, RF0028, RF0038, RF0039, RF0040, RF0041, RF0042, RF0050
- **Descrição:** Somente recursos armazenados na infraestrutura de mídia configurada pelo CoMangá podem ser retornados como capa. A URL pública deve ser derivada da chave interna e da configuração vigente. Obra, Edição e Volume não podem ser cadastrados nem permanecer associados sem uma capa interna válida. O fallback padrão é permitido exclusivamente quando um recurso interno já associado estiver temporariamente indisponível; ele não autoriza registro sem capa nem permite recorrer à URL externa de procedência.

## 5. Requisitos Não Funcionais:

### 5.1 Categoria: Desempenho

#### RNF01 — Desempenho de tempo de resposta do motor de busca de Obras e Edições

- **Categoria:** Desempenho
- **Prioridade:** Essencial
- **Descrição:** O sistema deve garantir alto desempenho e fluidez nas operações de leitura voltadas ao usuário final. O motor de busca de Obras e Edições deve processar a requisição e retornar os resultados paginados com um tempo de resposta máximo de 500 milissegundos (ms). Para fins de aferição de qualidade, esta métrica aplica-se ao percentil 95 (P95) do volume total de requisições, estabelecendo que 95% das buscas devem ser concluídas dentro deste limite de 500ms sob carga normal de operação. O escopo desta métrica de latência contabiliza o tempo de processamento interno do servidor (execução da query no banco de dados e serialização no Back-end) operando sob a premissa de uma conexão de rede cliente-servidor estável, com latência máxima de até 100ms. O sistema fica isento do descumprimento desta métrica caso os atrasos sejam comprovadamente decorrentes de oscilações, gargalos ou degradação na infraestrutura de internet do usuário final (ISP).

#### RNF02 — Capacidade de vazão e concorrência da API Pública

- **Categoria:** Desempenho
- **Prioridade:** Essencial
- **Descrição:** O sistema deve suportar picos de tráfego garantindo uma vazão de processamento de até 20 requisições por segundo (RPS) simultâneas nas rotas de consulta pública (Vitrine). Sob este teto de carga de estresse, o sistema deve manter uma taxa de falhas (erros HTTP 5xx) inferior a 1%.

#### RNF03 — Eficiência de tráfego e limite de paginação

- **Categoria:** Desempenho
- **Prioridade:** Essencial
- **Descrição:** O sistema deve otimizar o consumo de memória do servidor e a banda de rede do cliente impondo paginação compulsória em todas as operações de listagem de coleções (como o catálogo de Obras). O Payload de resposta (JSON) do Back-end não deve ultrapassar 2 Megabytes (MB) de tamanho absoluto por requisição. Para garantir este limite, a quantidade máxima de registros retornados por página fica estritamente limitada a 50 itens.

#### RNF04 — Desempenho de entrega de mídia estática (Capas)

- **Categoria:** Desempenho
- **Prioridade:** Essencial
- **Descrição:** O sistema deve persistir no PostgreSQL somente chaves internas e metadados das capas, nunca conteúdo binário ou base64. Os arquivos devem permanecer em armazenamento de objetos controlado pelo CoMangá e ser entregues por HTTPS mediante URLs derivadas da configuração interna. A interface deve usar dimensões estáveis, carregamento eficiente e fallback para recursos ausentes ou indisponíveis.

### 5.2. Categoria: Segurança

#### RNF05 — Armazenamento irreversível de credenciais (Hashing de Senhas)

- **Categoria:** Segurança
- **Prioridade:** Essencial
- **Descrição:** O sistema deve garantir a confidencialidade absoluta das credenciais de autenticação dos usuários. É estritamente proibida a persistência de senhas em texto plano (plain text) ou a utilização de qualquer método de criptografia reversível no banco de dados relacional. Todas as senhas de acesso devem obrigatoriamente ser processadas por um algoritmo de hash criptográfico unidirecional com geração de salt dinâmico antes da gravação. Para eliminar ambiguidades e prevenir o uso de padrões obsoletos ou vulneráveis a ataques de força bruta (como MD5 ou SHA-1), a equipe de desenvolvimento deve adotar exclusivamente o algoritmo Bcrypt com um fator de trabalho (Salt Rounds) mínimo de 10.

#### RNF06 — Gerenciamento de sessão stateful e transporte seguro

- **Categoria:** Segurança
- **Prioridade:** Essencial
- **Descrição:** O sistema deve operar com sessões stateful persistidas e validadas no servidor, permitindo revogação imediata no logout, na exclusão da conta e em outras operações sensíveis. O cliente deve receber apenas um identificador opaco por cookie HttpOnly; o servidor deve persistir somente o hash desse identificador. Em produção, o cookie deve utilizar Secure e uma política SameSite compatível com a topologia de implantação. Quando frontend e backend estiverem em origens distintas, a configuração deve permitir credenciais entre origens autorizadas sem ampliar indevidamente o CORS. A API deve aceitar credenciais apenas das origens explicitamente autorizadas e nunca deve transportar a sessão por localStorage ou cabeçalho Authorization.

#### RNF07 — Controle de Acesso Baseado em Funções (RBAC) via Middleware

- **Categoria:** Segurança
- **Prioridade:** Essencial
- **Descrição:** O sistema deve garantir que o acesso aos recursos e rotas administrativas seja estritamente limitado aos usuários com os privilégios adequados. A validação de autorização (RBAC) deve ocorrer obrigatoriamente na camada de interceptação (Middleware) da API, configurando um mecanismo de defesa preemptiva (Fail-Fast). O Middleware deve ler a role (função/papel) do usuário vinculada à sessão ativa recuperada do armazenamento Stateful. Caso o usuário autenticado não possua o nível de permissão exigido para a operação, o sistema deve abortar a requisição e retornar o código de erro HTTP 403 (Forbidden) instantaneamente, bloqueando o fluxo de execução antes que qualquer regra de negócio ou transação de banco de dados seja acionada.

#### RNF08 — Limitação de Taxa de Autenticação (Rate Limiting)

- **Categoria:** Segurança
- **Prioridade:** Essencial
- **Descrição:** O sistema deve proteger os pontos de entrada de credenciais (rotas de Login) contra ataques automatizados de força bruta e Credential Stuffing. Para isso, a API deve implementar um mecanismo estrito de Rate Limiting baseado na identificação do endereço IP de origem do cliente. O sistema deve permitir um limite máximo de 5 (cinco) tentativas de autenticação falhas consecutivas dentro de uma janela de tempo deslizante de 5 minutos. Ao atingir ou exceder este limiar, o Middleware de segurança deve interceptar e bloquear imediatamente quaisquer novas requisições de login oriundas daquele IP, abortando o processamento sem consultar o banco de dados e retornando obrigatoriamente o código de erro HTTP 429 (Too Many Requests). A restrição só deve ser levantada após a expiração natural da janela de penalidade.

### 5.3. Categoria: Confiabilidade

#### RNF09 — Integridade Transacional e Conformidade ACID

- **Categoria:** Confiabilidade
- **Prioridade:** Essencial
- **Descrição:** O sistema deve assegurar a integridade e a consistência absoluta dos dados em todas as operações de escrita que envolvam múltiplas entidades ou tabelas relacionadas (como a criação de uma Obra vinculada a suas Edições e Autores). Toda persistência multi-entidade deve ser executada obrigatoriamente sob o modelo de transação ACID (Atomicidade, Consistência, Isolamento e Durabilidade), garantindo que a operação seja tratada como uma unidade lógica atômica. Caso ocorra qualquer falha técnica ou violação de regra de integridade em qualquer etapa do processamento, o sistema deve realizar um Rollback total e automático, revertendo todas as alterações parciais e impedindo a persistência de registros corrompidos ou inconsistentes no banco de dados.

#### RNF10 — Integridade relacional rigorosa via constraints de banco

- **Categoria:** Confiabilidade
- **Prioridade:** Essencial
- **Descrição:** O sistema deve assegurar a integridade referencial por chaves estrangeiras, constraints e índices. As relações hierárquicas Obra → Edição → Volume e as referências a valores administrativos devem utilizar ON DELETE RESTRICT quando o registro dependente possuir ciclo de vida próprio. ON DELETE CASCADE deve ser reservado a sessões, vínculos associativos e demais registros sem existência independente, como os vínculos removidos com a exclusão permanente de uma conta.

#### RNF11 — Continuidade de dados e recuperação de desastres (DRP)

- **Categoria:** Confiabilidade
- **Prioridade:** Importante
- **Descrição:** O projeto deve manter uma política documentada de continuidade e recuperação de dados compatível com os recursos disponíveis no provedor PostgreSQL em nuvem. RPO alvo de 24 horas: a estratégia de backup deve limitar a perda máxima tolerável a um dia. RTO alvo de 4 horas: o procedimento de restauração deve buscar o retorno da operação em até quatro horas após a confirmação do incidente. Retenção mínima alvo de 7 dias: snapshots do provedor ou backups lógicos equivalentes devem preservar versões recuperáveis durante esse período. As limitações do plano contratado devem ser registradas. Quando o plano gratuito não oferecer os recursos necessários, a equipe deve complementar a política com exportações lógicas e testes periódicos de restauração.

#### RNF12 — Observabilidade e Tratamento de Exceções (Graceful Failure)

- **Categoria:** Confiabilidade
- **Prioridade:** Essencial
- **Descrição:** O sistema deve implementar uma camada global de tratamento de erros (Global Error Handler) que impeça o encerramento abrupto do processo (crash) diante de exceções não previstas. Métrica de Resposta: 100% das falhas de requisição (erros 4XX e 5XX) devem retornar um objeto JSON padronizado ao cliente, contendo uma mensagem amigável e um código de erro, nunca expondo o stack trace (rastro do código) em ambiente de produção. Métrica de Registro (Log): Todas as exceções de servidor devem ser capturadas e registradas em um serviço de observabilidade ou sistema de logs estruturados (ex: Console formatado ou arquivo rotativo), permitindo a rastreabilidade do erro (qual rota, qual horário e qual a mensagem técnica) para diagnóstico posterior.

### 5.4. Categoria: Usabilidade

#### RNF13 — Feedback semântico e comunicação de estado (Toasts)

- **Categoria:** Usabilidade
- **Prioridade:** Importante
- **Descrição:** O sistema deve fornecer feedback visual imediato e semântico para todas as ações executadas pelo usuário. Este feedback deve ser implementado via notificações flutuantes (Toasts) padronizadas: Mapeamento de Sucesso (Verde): Para operações concluídas com sucesso (ex: "Obra cadastrada"), com tempo de exibição de 3 segundos. Mapeamento de Erro (Vermelho): Para falhas de sistema ou validação, isolando o usuário de mensagens técnicas do compilador ou do banco. A mensagem deve ser traduzida para linguagem natural (ex: "Não foi possível conectar ao servidor") e permanecer em tela por 5 segundos ou até o fechamento manual. Acessibilidade: As cores devem possuir contraste adequado (padrão WCAG) e as notificações devem ser compatíveis com leitores de tela (atributo aria-live).

#### RNF14 — Adaptabilidade de interface e navegação responsiva (Mobile-First)

- **Categoria:** Usabilidade
- **Prioridade:** Importante
- **Descrição:** O sistema deve adaptar sua arquitetura de navegação e layout de forma fluida conforme a largura de tela do dispositivo de acesso, priorizando a experiência em dispositivos móveis (Mobile-First Strategy). A transição de layout deve obedecer aos seguintes critérios técnicos: Limiar de Transição (Breakpoint): O ponto de quebra para alteração da navegação principal é fixado em 768px. Contexto Mobile (< 768px): A navegação principal deve ser obrigatoriamente realizada via Bottom Navigation Bar (Barra Inferior), otimizando a ergonomia para o alcance do polegar (Thumb Zone). Contexto Desktop (≥ 768px): A navegação deve transitar para uma Sidebar (Barra Lateral) persistente ou retrátil, aproveitando o espaço horizontal para exibição de rótulos e submenus. Estratégia de Estilos: O CSS deve ser estruturado de forma que os estilos base sejam para dispositivos móveis, utilizando Media Queries apenas para expandir o layout para telas maiores, garantindo performance de renderização em dispositivos de menor capacidade.

#### RNF15 — Padronização geométrica e resiliência visual de capas

- **Categoria:** Usabilidade
- **Prioridade:** Importante
- **Descrição:** O sistema deve garantir a harmonia estrutural e a estabilidade do layout na Vitrine/Catálogo (especialmente no componente MangaCard), independentemente da resolução, tamanho ou proporção original das imagens cadastradas. Para proteger a interface contra quebras de grid e variações abruptas de layout (Cumulative Layout Shift), o sistema deve aplicar obrigatoriamente as seguintes diretrizes na renderização de capas: Proporção Geométrica: Uso estrito da propriedade CSS aspect-ratio: 2/3 para refletir o formato físico tradicional dos volumes impressos. Preenchimento: Uso obrigatório de object-fit: cover para garantir o preenchimento total do contêiner designado, realizando o corte automático das bordas excedentes sem distorcer (stretch) a obra original. Estado de Fallback (Resiliência): Caso o recurso interno da imagem falhe ao carregar (erro 404) ou a capa esteja ausente, o componente deve exibir imediatamente um estado de placeholder visualmente agradável (como um gradiente cinza neutro ou uma imagem padrão com a logomarca do sistema), mantendo o aspect-ratio intacto e informando visualmente que a capa original está indisponível.

#### RNF16 — Design semântico de Estados Vazios (Empty States)

- **Categoria:** Usabilidade
- **Prioridade:** Importante
- **Descrição:** Toda visualização que possa retornar zero registros deve apresentar um estado vazio explícito, sem deixar tabelas ou áreas de conteúdo sem explicação. O estado vazio deve conter uma mensagem diagnóstica em linguagem natural e, quando existir uma ação útil, um botão ou link para o próximo passo, como limpar filtros ou cadastrar um registro. O uso de ícone ou ilustração é opcional e deve respeitar o padrão visual da interface.
