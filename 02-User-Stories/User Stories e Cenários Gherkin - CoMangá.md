# User Stories e Cenários Gherkin do CoMangá

## Como ler este documento

- Cada seção corresponde a um requisito funcional do SERS (`01-SERS/SERS - CoMangá - Revisado.md`) e usa o mesmo identificador no título. As regras de negócio citadas entre parênteses (RN) são as do SERS.
- Cada `Feature` recebe uma etiqueta de estado:
  - `@vigente`: critério de aceite aplicável à etapa atual; a etiqueta não comprova implementação completa nem execução aprovada. Divergências conhecidas remetem à seção 6.3 do SERS.
  - `@futuro`: critérios de aceite de uma funcionalidade planejada (Calendário, Estante Digital e Lista de Desejos), que ainda não descrevem o sistema atual.
  - `@cancelado`: cenário de funcionalidade abandonada, mantido apenas como registro.
- As mensagens entre aspas são os textos exatos exibidos ou retornados hoje. Divergências conhecidas são apontadas por um comentário `#` e descritas somente na seção 6.3 do SERS.
- Os exemplos usam dados fictícios.

## RF0001: Cadastrar conta de acesso e disparar e-mail com link de ativação

**História de usuário:** Como visitante, quero cadastrar uma conta informando meus dados básicos, para receber o link de ativação e passar a usar as funcionalidades da conta.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Cadastrar conta de acesso e disparar e-mail com link de ativação

Scenario: Caminho Feliz - Cadastro de nova conta e envio do e-mail de ativação (RN0005, RN0006, RN0007)
Given que eu sou um visitante na tela de cadastro
And o e-mail "novo@exemplo.com" e o nome de usuário "novo_user" não estão em uso
When eu preencho Nome de Usuário com "novo_user", Data de Nascimento com "1990-08-15" e E-mail com "novo@exemplo.com"
And preencho Senha e Confirmação de Senha com "SenhaForte123!"
And envio o formulário de cadastro
Then o sistema deve informar "Conta criada com sucesso! Enviamos o e-mail de ativação."
And a conta deve ficar com o status "Pendente", apenas com o perfil "Usuário Padrão" e com a preferência de conteúdo adulto desativada
And "novo@exemplo.com" deve receber um e-mail com o link de ativação

Scenario: Caminho Alternativo 1 - Nome de Usuário fora do formato (RN0001)
Given que eu estou na tela de cadastro
When eu preencho o Nome de Usuário com "User Inválido!" e os demais campos corretamente
And envio o formulário
Then o cadastro deve ser recusado
And o campo de usuário deve exibir "Utilize entre 3 e 20 caracteres, sem espaços, acentos ou caracteres especiais."

Scenario: Caminho Alternativo 2 - Senha fraca (RN0002)
Given que eu estou na tela de cadastro
When eu preencho Senha e Confirmação de Senha com "fraca12" e os demais campos corretamente
And envio o formulário
Then o cadastro deve ser recusado
And o campo de senha deve exibir "Utilize no mínimo 8 caracteres, incluindo pelo menos uma letra maiúscula, uma minúscula, um número e um caractere especial."

Scenario: Caminho Alternativo 3 - Senha acima de 72 bytes (RN0002)
Given que eu estou na tela de cadastro
When eu informo uma senha forte que ocupa mais de 72 bytes em UTF-8 por conter acentos ou emojis
And envio o formulário
Then o cadastro deve ser recusado
And deve ser exibido "A senha deve ter no máximo 72 bytes em UTF-8; acentos e emojis podem ocupar mais de um byte."

Scenario: Caminho Alternativo 4 - E-mail já cadastrado (RN0003)
Given que o e-mail "existente@exemplo.com" já pertence a uma conta
When eu tento me cadastrar com "existente@exemplo.com"
Then o cadastro deve ser recusado
And o campo de e-mail deve exibir "Este endereço de e-mail já está em uso. Tente fazer login ou recuperar sua senha."

Scenario: Caminho Alternativo 5 - Nome de usuário já cadastrado (RN0004)
Given que o nome de usuário "user_existente" já pertence a uma conta
When eu tento me cadastrar com "user_existente"
Then o cadastro deve ser recusado
And o campo de usuário deve exibir "Este nome de usuário não está disponível. Por favor, escolha outro."

Scenario: Caminho Alternativo 6 - Confirmação de senha divergente (RN0020)
Given que eu estou na tela de cadastro
When eu preencho Senha com "SenhaForte123!" e Confirmação de Senha com "SenhaDiferente123!"
And envio o formulário
Then o cadastro deve ser recusado
And deve ser exibido "Divergência nos valores da senha e confirmação de senha!"

Scenario: Caminho Alternativo 7 - Data de nascimento futura ou inválida (RN0048)
Given que eu estou na tela de cadastro
When eu preencho a Data de Nascimento com "2099-01-01"
And envio o formulário
Then o cadastro deve ser recusado sem criar a conta
And deve ser informado que a data de nascimento precisa ser válida e não futura

Scenario: Caminho Alternativo 8 - Falha no envio do e-mail de ativação
Given que o serviço de e-mail está indisponível
When eu concluo um cadastro válido
Then a conta deve ser criada com o status "Pendente"
And o sistema deve informar "Conta criada, mas não foi possível enviar o e-mail de ativação. Use a opção de reenvio."
```

## RF0002: Ativar conta de acesso via token de ativação

**História de usuário:** Como dono de uma conta pendente, quero ativá-la pelo link recebido por e-mail, para confirmar minha identidade e poder entrar no sistema.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Ativar conta de acesso via token de ativação

Scenario: Caminho Feliz - Ativação da conta
Given que eu possuo uma conta "Pendente" e um link de ativação gerado há menos de 24 horas
When eu abro o link de ativação
Then o sistema deve exibir "Conta ativada com sucesso!"
And a conta deve passar ao status "Ativada"

Scenario: Caminho Alternativo 1 - Link expirado (RN0007)
Given que o meu link de ativação foi gerado há mais de 24 horas
When eu abro o link de ativação
Then a ativação deve ser recusada
And deve ser exibido "Este link de ativação expirou. Solicite um novo e-mail de ativação."

Scenario: Caminho Alternativo 2 - Link já utilizado ou inexistente (RN0008)
Given que o meu link de ativação já foi usado com sucesso
When eu abro o mesmo link novamente
Then a ativação deve ser recusada
And deve ser exibido "Link de ativação inválido!"
```

## RF0003: Reenviar e-mail com link de ativação da conta de acesso

**História de usuário:** Como dono de uma conta pendente, quero pedir um novo link de ativação, para concluir a ativação quando o link anterior se perder ou expirar.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Reenviar e-mail com link de ativação da conta de acesso

Scenario: Caminho Feliz - Reenvio do link de ativação
Given que eu possuo uma conta com o status "Pendente"
When eu informo o meu e-mail na tela de reenvio e envio a solicitação
Then o sistema deve exibir "Novo link de ativação enviado com sucesso para o seu e-mail!"
And eu devo receber um novo e-mail com o link de ativação

Scenario: Caminho Alternativo 1 - E-mail não cadastrado (RN0009)
Given que o e-mail informado não pertence a nenhuma conta
When eu solicito o reenvio
Then a solicitação deve ser recusada
And deve ser exibido "Endereço de e-mail não cadastrado"

Scenario: Caminho Alternativo 2 - Conta já ativada (RN0010)
Given que o e-mail informado pertence a uma conta "Ativada"
When eu solicito o reenvio
Then a solicitação deve ser recusada
And deve ser exibido "Este endereço de e-mail pertence a uma conta ativada."

Scenario: Caminho Alternativo 3 - Link anterior invalidado pelo reenvio (RN0011)
Given que eu solicitei o reenvio e recebi um novo link
When eu abro o link enviado no primeiro e-mail
Then a ativação deve ser recusada
And deve ser exibido "Link de ativação inválido!"
```

## RF0004: Autenticar credenciais da conta de acesso e retornar sessão de acesso

**História de usuário:** Como dono de uma conta ativada, quero entrar com e-mail e senha, para iniciar uma sessão segura no perfil que eu usei por último.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Autenticar credenciais da conta de acesso e retornar sessão de acesso

Scenario: Caminho Feliz - Login de conta ativada
Given que eu possuo uma conta "Ativada"
When eu informo e-mail e senha corretos na tela de login
Then o sistema deve exibir "Login realizado com sucesso!"
And eu devo ser levado à minha página de perfil com uma sessão ativa

Scenario: Caminho Feliz - Sessão inicia com o perfil preferido (RN0062)
Given que a minha conta possui os perfis "Administrador" e "Usuário Padrão"
And na última sessão eu escolhi o perfil "Administrador"
When eu faço login novamente
Then a nova sessão deve começar com o perfil ativo "Administrador"

Scenario: Caminho Alternativo 1 - Credenciais inválidas (RN0012)
Given que eu estou na tela de login
When eu informo um e-mail ou uma senha que não correspondem a uma conta
Then o login deve ser recusado
And deve ser exibido "Credenciais inválidas!"

Scenario: Caminho Alternativo 2 - Conta pendente (RN0013)
Given que a minha conta está com o status "Pendente"
When eu informo e-mail e senha corretos
Then o login deve ser recusado
And deve ser exibido "Conta de acesso pendente. Ative a conta com o e-mail de verificação enviado anteriormente."

Scenario: Caminho Alternativo 3 - Conta bloqueada (RN0013)
Given que a minha conta está com o status "Bloqueada"
When eu informo e-mail e senha corretos
Then o login deve ser recusado
And deve ser exibido "Esta conta foi bloqueada por razões de segurança."

Scenario: Caminho Alternativo 4 - Excesso de tentativas falhas (RNF08)
Given que houve 5 tentativas de login falhas do meu endereço de rede nos últimos 5 minutos
When eu tento entrar novamente, mesmo com a senha correta
Then a API deve recusar o login com HTTP 429 e a mensagem "Muitas tentativas de login. Tente novamente em alguns minutos."
# Divergência conhecida: ver SERS 6.3 (RNF08).

Scenario: Caminho Alternativo 5 - Acesso a área restrita sem sessão válida (RN0014)
Given que a minha sessão foi encerrada por logout ou por redefinição de senha
When eu tento abrir uma página ou recurso que exige autenticação
Then o acesso deve ser recusado
And deve ser exibido "Sua sessão é inválida ou foi encerrada. Por favor, faça login novamente."
```

## RF0005: Encerrar sessão de acesso ativa

**História de usuário:** Como usuário autenticado, quero sair da minha conta, para impedir acessos indevidos no meu dispositivo.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Encerrar sessão de acesso ativa

Scenario: Caminho Feliz - Logout voluntário
Given que eu estou autenticado na minha página de perfil
When eu escolho sair
Then o sistema deve exibir somente "Sessão encerrada com segurança."
And não deve exibir ao mesmo tempo o aviso de sessão inválida
And eu devo ser levado à tela de login
And as áreas restritas devem exigir novo login

Scenario: Caminho Alternativo 1 - Logout com falha na API
Given que eu estou autenticado
And a API não responde ao pedido de logout
When eu escolho sair
Then a Web deve apagar a sessão local, exibir "Sessão encerrada com segurança." e levar à tela de login

Scenario: Caminho Alternativo 2 - Pedido de logout sem sessão válida
Given que o pedido não traz uma sessão válida
When o logout é solicitado à API
Then a API deve recusar com "Sua sessão é inválida ou foi encerrada. Por favor, faça login novamente."
```

## RF0006: Solicitar redefinição de senha de acesso via e-mail

**História de usuário:** Como dono de uma conta, quero pedir a redefinição da senha pelo meu e-mail, para recuperar o acesso sem que terceiros descubram se o e-mail está cadastrado.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Solicitar redefinição de senha de acesso via e-mail

Scenario: Caminho Feliz - Conta ativada recebe o link de redefinição
Given que eu possuo uma conta "Ativada"
And a solicitação usa e-mail com formato válido e está dentro do limite permitido
When eu informo o meu e-mail na tela de recuperação de senha
Then o sistema deve exibir "Se houver uma conta apta para este e-mail, enviaremos as instruções de recuperação."
And eu devo receber um e-mail com o link de redefinição válido por 1 hora

Scenario: Caminho Alternativo 1 - E-mail inexistente recebe a mesma resposta (RN0015)
Given que a solicitação usa e-mail com formato válido e está dentro do limite permitido
When eu informo um e-mail que não pertence a nenhuma conta
Then o sistema deve exibir "Se houver uma conta apta para este e-mail, enviaremos as instruções de recuperação."
And nenhum e-mail deve ser enviado

Scenario Outline: Caminho Alternativo 2 - Conta não elegível recebe a mesma resposta (RN0016)
Given que a minha conta está com o status "<status>"
And a solicitação usa e-mail com formato válido e está dentro do limite permitido
When eu solicito a redefinição de senha
Then o sistema deve exibir "Se houver uma conta apta para este e-mail, enviaremos as instruções de recuperação."
And nenhum e-mail deve ser enviado
And o status da conta deve continuar "<status>"

Examples:
  | status    |
  | Pendente  |
  | Bloqueada |

Scenario: Caminho Alternativo 3 - Novo pedido em menos de 60 segundos (RN0015)
Given que eu recebi um link de redefinição há menos de 60 segundos
And a solicitação usa e-mail com formato válido e está dentro do limite permitido
When eu solicito a redefinição novamente
Then o sistema deve exibir a mesma resposta neutra
And nenhum novo e-mail deve ser enviado

Scenario: Caminho Alternativo 4 - Falha no envio do e-mail (RN0015)
Given que o serviço de e-mail está indisponível
And a solicitação usa e-mail com formato válido e está dentro do limite permitido
When eu solicito a redefinição para uma conta ativada
Then o sistema deve exibir a mesma resposta neutra, sem indicar a falha

Scenario: Caminho Alternativo 5 - E-mail com formato inválido
When eu informo "email-invalido" e envio a solicitação
Then o campo de e-mail deve indicar o erro de formato sem enviar a solicitação

Scenario: Caminho Alternativo 6 - Excesso de solicitações (RNF08)
Given que o meu endereço de rede fez 5 solicitações de recuperação nos últimos 15 minutos
When eu faço uma nova solicitação
Then ela deve ser recusada com "Muitas solicitações. Tente novamente em alguns minutos."
```

## RF0007: Redefinir senha de acesso via token de redefinição

**História de usuário:** Como dono de uma conta que esqueceu a senha, quero definir uma nova senha pelo link recebido, para voltar a entrar na conta.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Redefinir senha de acesso via token de redefinição

Scenario: Caminho Feliz - Redefinição com link válido
Given que eu abri um link de redefinição gerado há menos de 1 hora
When eu preencho Nova Senha e Confirmação com "SenhaForte123!" e envio
Then o sistema deve exibir "Senha redefinida com sucesso. Faça login novamente."
And eu devo ser levado à tela de login
And todas as sessões abertas da minha conta devem ser encerradas

Scenario: Caminho Alternativo 1 - Nova senha fraca (RN0002)
Given que eu abri um link de redefinição válido
When eu preencho Nova Senha e Confirmação com "fraca12"
Then a redefinição deve ser recusada
And deve ser exibido "Utilize no mínimo 8 caracteres, incluindo pelo menos uma letra maiúscula, uma minúscula, um número e um caractere especial."

Scenario: Caminho Alternativo 2 - Link expirado (RN0017)
Given que o meu link de redefinição foi gerado há 1 hora ou mais
When eu envio uma nova senha válida
Then a redefinição deve ser recusada
And deve ser exibido "Este link de redefinição expirou. Solicite a redefinição novamente."

Scenario: Caminho Alternativo 3 - Link já utilizado (RN0018)
Given que o meu link de redefinição já foi usado com sucesso
When eu tento usá-lo novamente
Then a redefinição deve ser recusada
And deve ser exibido "Link de redefinição inválido!"

Scenario: Caminho Alternativo 4 - Link substituído por um pedido mais recente (RN0019)
Given que eu pedi a redefinição duas vezes e recebi dois links
When eu uso o primeiro link
Then a redefinição deve ser recusada
And deve ser exibido "Link de redefinição inválido!"

Scenario: Caminho Alternativo 5 - Confirmação divergente (RN0020)
Given que eu abri um link de redefinição válido
When eu preencho Nova Senha com "SenhaForte123!" e Confirmação com "SenhaDiferente123!"
Then a redefinição deve ser recusada sem consultar a API
And o campo de confirmação deve exibir "As senhas não conferem."
```

## RF0008: Redefinir senha de acesso em sessão ativa

**História de usuário:** Como usuário autenticado, quero trocar minha senha nas configurações avançadas do perfil, para manter a conta segura sem sair do dispositivo que estou usando.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Redefinir senha de acesso em sessão ativa

Scenario: Caminho Feliz - Troca de senha mantém a sessão atual
Given que eu estou autenticado e tenho outra sessão aberta em outro dispositivo
When eu informo a Senha Atual correta e preencho Nova Senha e Confirmação com "NovaSenhaForte123!"
Then o sistema deve informar "Senha alterada com sucesso! As demais sessões da conta foram encerradas."
And os campos devem ser limpos e eu devo continuar autenticado na página de perfil
And a sessão do outro dispositivo deve exigir novo login

Scenario: Caminho Alternativo 1 - Senha atual incorreta (RN0021)
Given que eu estou autenticado
When eu informo uma Senha Atual incorreta e uma nova senha válida
Then a troca deve ser recusada
And deve ser exibido "Senha atual incorreta!"

Scenario: Caminho Alternativo 2 - Nova senha fraca (RN0002)
Given que eu estou autenticado
When eu informo a Senha Atual correta e a nova senha "fraca12"
Then a troca deve ser recusada
And deve ser exibido "Utilize no mínimo 8 caracteres, incluindo pelo menos uma letra maiúscula, uma minúscula, um número e um caractere especial."

Scenario: Caminho Alternativo 3 - Confirmação divergente (RN0020)
Given que eu estou autenticado
When eu preencho Nova Senha com "NovaSenhaForte123!" e Confirmação com "DiferenteSenha123!"
Then a troca deve ser recusada
And deve ser exibido "Divergência nos valores da senha e confirmação de senha!"

Scenario: Caminho Alternativo 4 - Nova senha igual à atual (RN0073)
Given que eu estou autenticado
When eu informo como nova senha a mesma senha atual
Then a troca deve ser recusada
And deve ser exibido "A nova senha deve ser diferente da senha atual."

Scenario: Caminho Alternativo 5 - Excesso de tentativas com a senha atual (RNF08)
Given que a minha conta teve 5 tentativas com senha atual incorreta nos últimos 5 minutos
When eu tento trocar a senha novamente
Then a troca deve ser recusada
And deve ser exibido "Muitas tentativas com a senha atual. Tente novamente em alguns minutos."
```

## RF0009: Consultar dados cadastrais da própria conta

**História de usuário:** Como usuário autenticado, quero ver os dados da minha conta, para conferir minhas informações, meus perfis e minhas preferências.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar dados cadastrais da própria conta

Scenario: Caminho Feliz - Página de perfil exibe os dados da conta
Given que eu estou autenticado
When eu abro a minha página de perfil
Then devo ver meu nome de usuário e meu e-mail
And devo ver meus perfis e o perfil ativo (quando a conta possui mais de um perfil)
And a minha data de nascimento não deve ser exibida

Scenario: Caminho Alternativo 1 - Identificador de outra conta é ignorado (RN0022)
Given que eu estou autenticado com o perfil "Usuário Padrão"
When eu consulto os dados da conta informando o identificador de outra conta
Then o sistema deve retornar somente os dados da minha própria conta
And nenhum dado da outra conta deve ser exposto
```

## RF0010: Alterar nome de usuário em sessão ativa

**História de usuário:** Como usuário autenticado, quero alterar meu nome de usuário nas configurações avançadas, para atualizar minha identificação no sistema.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Alterar nome de usuário em sessão ativa

Scenario: Caminho Feliz - Alteração do nome de usuário
Given que eu estou autenticado
When eu informo o novo nome de usuário "novo_user_123" e salvo
Then o sistema deve exibir "Nome de usuário atualizado com sucesso!"
And eu devo continuar na página de perfil com o novo nome no cabeçalho

Scenario: Caminho Alternativo 1 - Formato inválido (RN0001)
Given que eu estou autenticado
When eu informo o novo nome "ab" ou "User Inválido!"
Then a alteração deve ser recusada sem consultar a API
And deve ser exibido "Utilize entre 3 e 20 caracteres, sem espaços, acentos ou caracteres especiais."

Scenario: Caminho Alternativo 2 - Nome já utilizado por outra conta (RN0004)
Given que o nome "user_existente" pertence a outra conta
When eu tento adotar o nome "user_existente"
Then a alteração deve ser recusada
And deve ser exibido "Este nome de usuário não está disponível. Por favor, escolha outro."

Scenario: Caminho Alternativo 3 - Repetição do nome atual
Given que eu estou autenticado como "novo_user_123"
When eu informo "novo_user_123" como novo nome
Then a Web deve recusar a alteração sem consultar a API

Scenario: Caminho Alternativo 4 - Identificador de outra conta é ignorado (RN0022)
Given que eu estou autenticado com o perfil "Usuário Padrão"
When eu envio a alteração de nome informando o identificador de outra conta
Then somente o nome da minha própria conta pode ser alterado
And a outra conta deve permanecer inalterada
```

## RF0011: Alterar preferência de exibição de conteúdo adulto

**História de usuário:** Como usuário maior de idade, quero ativar ou desativar a exibição de conteúdo adulto, para controlar o que aparece no catálogo.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Alterar preferência de exibição de conteúdo adulto

Scenario: Caminho Feliz - Maior de idade ativa a preferência
Given que eu estou autenticado com o perfil ativo "Usuário Padrão"
And minha data de nascimento indica 18 anos completos
When eu ativo a opção "Conteúdo +18" na página de perfil
Then o sistema deve exibir "Conteúdo +18 ativado."
And a opção deve passar a "Ativado"
And o catálogo público deve passar a exibir conteúdo adulto para mim

Scenario: Caminho Feliz - Desativação da preferência
Given que a minha preferência de conteúdo adulto está ativada
When eu desativo a opção "Conteúdo +18"
Then a opção deve passar a "Desativado"
And o conteúdo adulto deve deixar de aparecer para mim no catálogo público

Scenario: Caminho Alternativo 1 - Menor de idade não pode ativar (RN0048)
Given que eu estou autenticado e minha data de nascimento indica menos de 18 anos
When eu abro a página de perfil
Then a opção "Conteúdo +18" não deve ser exibida
And uma tentativa de ativação enviada diretamente deve ser recusada com "Conteúdo +18 exige data de nascimento informada e 18 anos completos."

Scenario: Caminho Alternativo 2 - Opção oculta com o perfil Administrador ativo (RN0048)
Given que eu possuo os perfis "Administrador" e "Usuário Padrão"
And o meu perfil ativo é "Administrador"
When eu abro a página de perfil
Then a opção "Conteúdo +18" não deve ser exibida
```

## RF0012: Excluir permanentemente conta de acesso e dados vinculados

**História de usuário:** Como usuário autenticado, quero excluir minha conta confirmando minha senha, para remover meus dados do sistema de forma definitiva.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Excluir permanentemente conta de acesso e dados vinculados

Scenario: Caminho Feliz - Exclusão da própria conta
Given que eu estou autenticado
When eu abro a opção de excluir conta, informo a Senha Atual correta e confirmo
Then o sistema deve exibir "Conta excluida permanentemente."
And a minha sessão deve ser encerrada
And não deve ser possível entrar novamente com essa conta

Scenario: Caminho Alternativo 1 - Senha atual incorreta (RN0021)
Given que eu estou autenticado
When eu confirmo a exclusão com uma Senha Atual incorreta
Then a exclusão deve ser recusada
And deve ser exibido "Senha atual incorreta!"

Scenario: Caminho Alternativo 2 - Último Administrador (RN0063)
Given que a minha conta é a única conta ativada com o perfil "Administrador"
When eu confirmo a exclusão com a senha correta
Then a exclusão deve ser recusada
And deve ser exibido "Operação bloqueada: o sistema ficaria sem nenhum administrador ativo."

Scenario: Caminho Alternativo 3 - Identificador de outra conta é ignorado (RN0022)
Given que eu estou autenticado com o perfil "Usuário Padrão"
When eu envio a exclusão informando o identificador de outra conta
Then somente a minha própria conta pode ser excluída
And a outra conta deve permanecer intacta
```

## RF0013: Cadastrar nova Obra no banco de dados

**História de usuário:** Como Administrador, quero cadastrar uma Obra com seus dados de identificação, autoria, publicação, classificação, capa e sinopse, para ampliar o catálogo e prepará-la para publicação.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Cadastrar nova Obra no banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"
And estou em "Novo mangá"

Scenario: Caminho Feliz - Cadastro pelas quatro etapas (RN0027)
When eu preencho em "Identificação" o título "Obra Exemplo", o título original "オブラ・エクサンプル", o título romanizado "Obra Ekusanpuru", o país "Japão" e o tipo "Mangá"
And em "Autoria" adiciono um autor com o papel "História e Arte"
And em "Publicação original e classificação" informo editora original, status "Completa", anos de início e fim, pré-publicação, demografia e gênero
And em "Capa e sinopse" importo a capa e depois informo a sinopse
And salvo o cadastro
Then a Obra deve ser cadastrada com a visibilidade "Privado"
And o sistema deve oferecer as próximas ações após o cadastro

Scenario: Caminho Feliz - Título original é opcional (RN0026)
When eu cadastro uma Obra válida sem título original
Then a Obra deve ser cadastrada
# Divergência conhecida: ver SERS 6.3 (RN0026).

Scenario: Caminho Alternativo 1 - Autor repetido (RN0023)
When eu adiciono o mesmo autor duas vezes
And tento salvar
Then o cadastro deve ser recusado com "Autor duplicado!"

Scenario Outline: Caminho Alternativo 2 - Papéis incompatíveis para o mesmo autor (RN0068)
When eu tento atribuir ao mesmo autor os papéis <papeis>
Then o formulário deve desabilitar o papel incompatível
And uma tentativa enviada diretamente à API deve ser recusada com "Preencha os campos obrigatórios da Obra."
# Divergência conhecida: ver SERS 6.3 (RN0068).

Examples:
  | papeis                         |
  | "História e Arte" e "História" |
  | "História e Arte" e "Arte"     |
  | "História" e "Arte"            |

Scenario: Caminho Alternativo 3 - Título já cadastrado (RN0025)
Given que já existe a Obra "Fire Force"
When eu tento cadastrar outra Obra com o título " fire force "
Then o cadastro deve ser recusado com "Obra já cadastrada!"

Scenario: Caminho Alternativo 4 - Dados obrigatórios ausentes (RN0026)
When eu tento avançar ou salvar sem um dado obrigatório da etapa, como o país de origem ou o título romanizado
Then os campos ausentes devem ser marcados com "Preencha o campo obrigatório."
And nada deve ser enviado à API
And um cadastro sem dado obrigatório enviado diretamente à API deve ser recusado com "Preencha os campos obrigatórios da Obra."

Scenario: Caminho Alternativo 5 - Tipo de Obra restrito ao país (RN0024)
When eu escolho o país "Coreia do Sul"
Then somente os tipos "Manhwa" e "Novel" devem ser oferecidos
And uma combinação incompatível enviada diretamente deve ser recusada com "Um ou mais valores selecionados são inválidos."

Scenario: Caminho Alternativo 6 - Hentai marca e bloqueia o conteúdo adulto no formulário (RN0067)
When eu seleciono o gênero "Hentai"
Then a opção de conteúdo adulto deve ficar marcada e bloqueada
When eu retiro o gênero "Hentai"
Then a opção de conteúdo adulto deve ser desmarcada automaticamente

Scenario: Caminho Alternativo 7 - A API força o conteúdo adulto com Hentai (RN0067)
When uma Obra com o gênero "Hentai" é cadastrada com o conteúdo adulto desativado
Then a Obra deve ser gravada como conteúdo adulto

Scenario: Caminho Alternativo 8 - Artbook e Databook são lançamento direto (RN0026)
When eu escolho o tipo "Artbook" ou "Databook"
Then o lançamento direto deve ficar marcado e bloqueado
And demografia e pré-publicação devem ficar desabilitadas e vazias

Scenario: Caminho Alternativo 9 - Publicação em andamento dispensa o ano de fim (RN0026)
When eu escolho o status "Em andamento" ou "Em hiato"
Then o ano de fim deve ser limpo e desabilitado

Scenario: Caminho Alternativo 10 - Demografia só para Mangá (RN0026)
When eu escolho um tipo diferente de "Mangá"
Then a demografia deve ficar desabilitada e vazia

@cancelado
Scenario: Número de volumes originais
# Cancelado: o campo foi removido por ser ambíguo entre mangás, novels, manhwas e outras formas de publicação.
```

## RF0014: Alterar dados de uma Obra específica no banco de dados

**História de usuário:** Como Administrador, quero corrigir ou completar os dados de uma Obra, para manter o catálogo preciso sem recadastrar o registro.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Alterar dados de uma Obra específica no banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"
And abri a edição da Obra "Obra Exemplo"

Scenario: Caminho Feliz - Alteração parcial
When eu altero apenas o status da publicação original para "Completa" e salvo
Then somente esse dado deve mudar
And os demais dados da Obra devem permanecer como estavam

Scenario: Caminho Feliz - Troca de título preserva a URL da Obra
When eu altero o título para "Obra Exemplo Nova Edição" e salvo
Then o novo título deve ser exibido
And a Obra deve continuar acessível pelo mesmo endereço de antes

Scenario: Caminho Alternativo 1 - Autor repetido (RN0023)
When eu incluo o mesmo autor duas vezes e salvo
Then a alteração deve ser recusada com "Autor duplicado!"
And a capa atual deve ser preservada

Scenario: Caminho Alternativo 2 - Título de outra Obra (RN0025)
Given que existe a Obra "Outra Obra"
When eu altero o título para "Outra Obra" e salvo
Then a alteração deve ser recusada com "Obra já cadastrada!"

Scenario: Caminho Alternativo 3 - Remoção de dado obrigatório (RN0026)
When eu apago o título em português e tento salvar
Then o campo deve ser marcado com "Preencha o campo obrigatório.", sem enviar a alteração
And uma alteração com o título vazio enviada diretamente à API deve ser recusada com "Preencha os campos obrigatórios da Obra."

Scenario: Caminho Alternativo 4 - Remover Hentai não desativa o conteúdo adulto na API (RN0067)
Given que a Obra tem o gênero "Hentai" e está marcada como conteúdo adulto
When a Obra é alterada diretamente pela API retirando "Hentai", sem informar o conteúdo adulto
Then a Obra deve continuar marcada como conteúdo adulto
When em seguida o conteúdo adulto é desativado explicitamente
Then a Obra deve deixar de ser conteúdo adulto

Scenario: Caminho Alternativo 5 - Desativar conteúdo adulto com Hentai associado (RN0067)
Given que a Obra tem o gênero "Hentai"
When a alteração envia o conteúdo adulto desativado
Then a Obra deve continuar marcada como conteúdo adulto

Scenario: Caminho Alternativo 6 - Troca de capa (RN0060)
When eu importo uma nova capa e salvo
Then a Obra deve passar a exibir a nova capa
And se a troca falhar, a capa anterior deve continuar exibida
```

## RF0015: Excluir uma Obra específica do banco de dados

**História de usuário:** Como Administrador, quero excluir uma Obra cadastrada por engano, para manter o catálogo limpo sem quebrar Edições existentes.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Excluir uma Obra específica do banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador" em "Gerenciar mangás"

Scenario: Caminho Feliz - Exclusão de Obra privada sem Edições
Given que a Obra é "Privado" e não tem Edições
When eu excluo a Obra e confirmo
Then a Obra deve deixar de existir no catálogo

Scenario: Caminho Alternativo 1 - Obra pública (RN0028)
Given que a Obra é "Público"
When eu tento excluí-la
Then a exclusão deve ser recusada com "Essa Obra está pública, não pode ser excluída!"

Scenario: Caminho Alternativo 2 - Obra com Edições (RN0029)
Given que a Obra é "Privado" e tem ao menos uma Edição
When eu tento excluí-la
Then a exclusão deve ser recusada com "Essa Obra possui Edições vinculadas, não pode ser excluída!"
And a Obra e as Edições devem permanecer

Scenario: Caminho Alternativo 3 - Cancelamento da confirmação
When eu inicio a exclusão e cancelo a confirmação
Then a Obra deve permanecer inalterada
```

## RF0016: Consultar coleção de Obras cadastradas no banco de dados

**História de usuário:** Como Administrador, quero listar, buscar, filtrar e ordenar as Obras, para localizar rapidamente os registros que preciso gerenciar.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar coleção de Obras cadastradas no banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"

Scenario: Caminho Feliz - Listagem padrão em "Gerenciar mangás"
When eu abro "Gerenciar mangás"
Then devo ver as Obras em ordem crescente de título, paginadas
And cada Obra deve mostrar capa, título, autores ordenados pelo crédito, país, tipo, quantidade de Edições e visibilidade

Scenario: Caminho Alternativo 1 - Busca por título e filtros combinados
When eu busco "Fire" e filtro por tipo "Mangá", país "Japão" e visibilidade "Público"
Then devo ver somente as Obras cujo título contém "Fire" e que atendem a todos os filtros

Scenario: Caminho Alternativo 2 - Ordenação invertida
When eu inverto a ordenação por título
Then as Obras devem aparecer em ordem decrescente de título

Scenario: Caminho Alternativo 3 - Nenhum resultado
When eu aplico uma busca sem correspondência
Then deve ser exibido um estado vazio específico da busca, sem erro

Scenario: Caminho Alternativo 4 - Alternância entre lista e grade
When eu alterno para a visualização em grade
Then as mesmas Obras devem ser exibidas em cartões, com fallback de capa quando necessário
```

## RF0017: Consultar dados de uma Obra específica do banco de dados

**História de usuário:** Como Administrador, quero ver todos os dados de uma Obra, para auditar o registro no seu estado mais recente.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar dados de uma Obra específica do banco de dados

Scenario: Caminho Feliz - Abertura da Obra pelo endereço administrativo
Given que eu estou autenticado com o perfil ativo "Administrador"
When eu abro "/admin/gerenciar-mangas/obras/obra-exemplo/edicoes"
Then devo ver a prévia completa da Obra "Obra Exemplo" com os dados mais recentes

Scenario: Caminho Alternativo 1 - Obra inexistente
Given que eu estou autenticado com o perfil ativo "Administrador"
When eu abro o endereço administrativo de uma Obra que não existe
Then deve ser exibido "Obra não encontrada."
```

## RF0018: Alterar status de visibilidade de uma Obra específica do banco de dados

**História de usuário:** Como Administrador, quero publicar ou despublicar uma Obra, para controlar sua disponibilidade no catálogo público.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Alterar status de visibilidade de uma Obra específica do banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"

Scenario: Caminho Feliz - Publicação da Obra
Given que a Obra é "Privado"
When eu altero a visibilidade para "Público"
Then a Obra deve ficar "Público"
And suas Edições devem manter a visibilidade que tinham

Scenario: Caminho Feliz - Despublicação sem Edições públicas
Given que a Obra é "Público" e nenhuma Edição dela é pública
When eu altero a visibilidade para "Privado"
Then a Obra deve ficar "Privado"

Scenario: Caminho Alternativo 1 - Obra com Edição pública (RN0030)
Given que a Obra é "Público" e tem uma Edição "Público"
When eu altero a visibilidade da Obra para "Privado"
Then a alteração deve ser recusada com "Essa Obra possui Edições públicas, não pode ser rebaixada para privada!"
```

## RF0019: Cadastrar nova Edição vinculada a uma Obra no banco de dados

**História de usuário:** Como Administrador, quero cadastrar uma Edição brasileira de uma Obra, para registrar cada publicação nacional com sua editora e suas características.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Cadastrar nova Edição vinculada a uma Obra no banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"
And abri o cadastro de Edição da Obra "Obra Exemplo"

Scenario: Caminho Feliz - Cadastro com todos os dados (RN0031, RN0034)
When eu informo número da edição 1, editora brasileira, status "Em andamento", acabamento, formato e os miolos "Offset" e "Couché"
And salvo
Then a Edição deve ser cadastrada na Obra com a visibilidade "Privado"
And os dois miolos devem ser preservados na ordem selecionada
And ela deve aparecer sem capa até existir o Volume 1

Scenario: Caminho Feliz - Cadastro sem metadados físicos conhecidos (RN0033)
When eu informo apenas número da edição, editora brasileira e status
And salvo
Then a Edição deve ser cadastrada com acabamento e formato nulos e a lista de miolos vazia

Scenario: Caminho Alternativo 1 - Número de edição repetido (RN0032)
Given que a Obra já tem a 1ª edição
When eu cadastro outra Edição com o número 1
Then o cadastro deve ser recusado com "Essa Obra já possui uma Edição com esse número cronológico!"

Scenario: Caminho Alternativo 2 - Dados obrigatórios ausentes (RN0033)
When eu tento salvar sem a editora brasileira ou sem o status no Brasil
Then o cadastro deve ser recusado e os campos obrigatórios devem ser indicados

Scenario: Caminho Alternativo 3 - Edição não tem capa própria (RN0069)
When eu abro o formulário de Edição
Then não deve haver campo de importação de capa

@cancelado
Scenario: Tipo de Edição
# Cancelado: o tipo de Edição (Tankobon, Kanzenban, 2 em 1...) foi removido do sistema.
```

## RF0020: Alterar dados de uma Edição específica no banco de dados

**História de usuário:** Como Administrador, quero corrigir ou completar os dados de uma Edição, para manter a publicação brasileira atualizada.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Alterar dados de uma Edição específica no banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"
And abri a edição da 2ª edição da Obra "Obra Exemplo"

Scenario: Caminho Feliz - Alteração parcial
When eu altero apenas o status no Brasil para "Completa" e salvo
Then somente esse dado deve mudar

Scenario: Caminho Feliz - Limpeza de metadado opcional (RN0033)
When eu removo todos os miolos selecionados e salvo
Then a Edição deve ficar com a lista de miolos vazia
And o catálogo público não deve exibir um valor inventado para o miolo

Scenario: Caminho Alternativo 1 - Número de outra Edição (RN0032)
Given que a Obra também tem a 1ª edição
When eu altero o número desta Edição para 1
Then a alteração deve ser recusada com "Essa Obra já possui uma Edição com esse número cronológico!"
```

## RF0021: Excluir uma Edição específica do banco de dados

**História de usuário:** Como Administrador, quero excluir uma Edição cadastrada por engano, sem deixar Volumes órfãos.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Excluir uma Edição específica do banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador" em "Gerenciar edições"

Scenario: Caminho Feliz - Exclusão de Edição privada sem Volumes
Given que a Edição é "Privado" e não tem Volumes
When eu excluo a Edição e confirmo
Then a Edição deve deixar de existir

Scenario: Caminho Alternativo 1 - Edição pública (RN0035)
Given que a Edição é "Público"
When eu tento excluí-la
Then a exclusão deve ser recusada com "Essa Edição está pública, não pode ser excluída!"

Scenario: Caminho Alternativo 2 - Edição com Volumes (RN0036)
Given que a Edição é "Privado" e tem Volumes
When eu tento excluí-la
Then a exclusão deve ser recusada com "Essa Edição possui Volumes vinculados, não pode ser excluída!"
```

## RF0022: Consultar coleção de Edições vinculadas a uma Obra no banco de dados

**História de usuário:** Como Administrador, quero ver as Edições de uma Obra, para acompanhar as publicações brasileiras e acessar seus Volumes.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar coleção de Edições vinculadas a uma Obra no banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"

Scenario: Caminho Feliz - "Gerenciar edições" (RN0051)
Given que a Obra "Obra Exemplo" tem a 1ª e a 2ª edição
When eu abro "Gerenciar edições" dessa Obra
Then devo ver a prévia da Obra e as Edições em ordem decrescente de número (2ª e depois 1ª)
And cada Edição deve mostrar a capa do seu Volume 1, o número em forma ordinal, a editora, a quantidade de Volumes e a visibilidade

Scenario: Caminho Alternativo 1 - Edição sem Volume 1 (RN0069)
Given que a 2ª edição tem apenas o Volume 0
When eu abro "Gerenciar edições"
Then a 2ª edição deve aparecer com "Sem capa (cadastre o Volume 1)", sem usar a capa do Volume 0

Scenario: Caminho Alternativo 2 - Obra sem Edições
Given que a Obra não tem Edições
When eu abro "Gerenciar edições"
Then deve ser exibido um estado vazio, sem erro
```

## RF0023: Consultar dados de uma Edição específica do banco de dados

**História de usuário:** Como Administrador, quero ver todos os dados de uma Edição, para auditar o registro no seu estado mais recente.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar dados de uma Edição específica do banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"

Scenario: Caminho Feliz - Consulta da Edição
When eu consulto uma Edição existente
Then devo receber todos os seus dados, a Obra vinculada e a capa derivada, no estado mais recente

Scenario: Caminho Alternativo 1 - Edição inexistente
When eu consulto uma Edição que não existe
Then deve ser informado "Edição não encontrada."

Scenario: Caminho Alternativo 2 - Identificador inválido
When eu consulto uma Edição com o identificador "abc"
Then deve ser informado "Formato de identificador invalido."
```

## RF0024: Alterar status de visibilidade de uma Edição específica do banco de dados

**História de usuário:** Como Administrador, quero publicar ou despublicar uma Edição e seus Volumes, para controlar o que aparece no catálogo público.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Alterar status de visibilidade de uma Edição específica do banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"

Scenario: Caminho Feliz - Publicação propaga para os Volumes (RN0040, RN0069)
Given que a Obra é "Público"
And a Edição é "Privado" e tem Volume 1 com capa
When eu altero a visibilidade da Edição para "Público"
Then a Edição e todos os seus Volumes devem ficar "Público"

Scenario: Caminho Feliz - Despublicação propaga para os Volumes (RN0040)
Given que a Edição é "Público"
When eu altero a visibilidade da Edição para "Privado"
Then a Edição e todos os seus Volumes devem ficar "Privado"

Scenario: Caminho Alternativo 1 - Obra privada (RN0037)
Given que a Obra é "Privado"
When eu tento publicar uma Edição dela
Then a publicação deve ser recusada com "Essa Edição está vinculada a uma Obra privada, não pode ser publicada!"

Scenario: Caminho Alternativo 2 - Edição sem Volume 1 (RN0069)
Given que a Obra é "Público" e a Edição não tem Volume 1
When eu tento publicar a Edição
Then a publicação deve ser recusada com "Essa Edição não possui o Volume 1 com capa interna válida, não pode ser publicada!"
And a Edição e seus Volumes devem continuar "Privado"

@futuro
Scenario: Caminho Alternativo 3 - Edição com Volumes em acervos pessoais (RN0042)
Given que algum Volume da Edição pública está em uma Estante Digital ou Lista de Desejos
When eu tento tornar a Edição "Privado"
Then a alteração deve ser recusada com "Essa Edição tem Volumes vinculados a usuários, não pode ser tornada privada!"
```

## RF0025: Cadastrar novo Volume vinculado a uma Edição no banco de dados

**História de usuário:** Como Administrador, quero cadastrar cada Volume de uma Edição com capa, data, preço e identificadores, para detalhar a publicação brasileira.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Cadastrar novo Volume vinculado a uma Edição no banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"
And abri o cadastro de Volume da 1ª edição da Obra "Obra Exemplo"

Scenario: Caminho Feliz - Cadastro com dados obrigatórios e opcionais
When eu informo o número 1, importo a capa e informo a data completa "20/08/2026"
And opcionalmente informo páginas, preço "79,90" em "R$", ISBN-10, ISBN-13, link de afiliado e sinopse
And salvo
Then o Volume deve ser cadastrado com a mesma visibilidade atual da Edição

Scenario Outline: Caminho Feliz - Precisão da data de lançamento (RN0039)
When eu informo a precisão "<precisao>" com <data>
Then o Volume deve ser cadastrado e exibir a data como "<exibicao>"

Examples:
  | precisao   | data             | exibicao   |
  | Completa   | dia 20, mês 8, 2026 | 20/08/2026 |
  | Mês e ano  | mês 8, 2026      | 08/2026    |
  | Ano        | 2026             | 2026       |

Scenario: Caminho Alternativo 1 - Número repetido na Edição (RN0038)
Given que a Edição já tem o Volume 1
When eu cadastro outro Volume com o número 1
Then o cadastro deve ser recusado com "Um volume desta edição com esse mesmo número já foi cadastrado anteriormente!"

Scenario: Caminho Alternativo 2 - Dados obrigatórios ausentes (RN0039)
When eu tento salvar sem capa ou sem data de lançamento
Then o cadastro deve ser recusado e os campos obrigatórios devem ser indicados

Scenario: Caminho Alternativo 3 - ISBN com dígito verificador inválido (RN0039)
When eu informo o ISBN-13 "9781234567891" com os demais dados válidos e salvo
Then o formulário deve enviar o cadastro sem validar o ISBN
And a API deve recusar o cadastro com "Preencha os campos obrigatórios do Volume."
# Divergência conhecida: ver SERS 6.3 (RN0039).

Scenario: Caminho Alternativo 4 - Volume zero é aceito (RN0039)
When eu informo o número 0 com os demais dados válidos
Then o Volume 0 deve ser cadastrado
```

## RF0026: Alterar dados de um Volume específico no banco de dados

**História de usuário:** Como Administrador, quero corrigir ou completar os dados de um Volume, para manter preço, lançamento e identificadores atualizados.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Alterar dados de um Volume específico no banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"

Scenario: Caminho Feliz - Alteração parcial
When eu altero apenas o preço e a sinopse do Volume 2 e salvo
Then somente esses dados devem mudar e a capa atual deve ser preservada

Scenario: Caminho Feliz - Redução da precisão da data (RN0039)
Given que o Volume tem a data completa "20/08/2026"
When eu altero a precisão para "Ano"
Then o Volume deve exibir apenas "2026"

Scenario: Caminho Alternativo 1 - Número de outro Volume (RN0038)
Given que a Edição tem os Volumes 1 e 2
When eu altero o número do Volume 2 para 1
Then a alteração deve ser recusada com "Um volume desta edição com esse mesmo número já foi cadastrado anteriormente!"

Scenario: Caminho Alternativo 2 - Data resultante inválida (RN0039)
When eu altero a data completa para "31/02/2026"
Then a alteração deve ser recusada com "Informe uma data de lançamento válida para o Volume."

Scenario: Caminho Alternativo 3 - Renumerar o Volume 1 de Edição pública (RN0069)
Given que a Edição é "Público"
When eu altero o número do Volume 1 para 10
Then a alteração deve ser recusada com "Esse é o Volume 1 de uma Edição pública: renumerá-lo deixaria a Edição sem capa!"
```

## RF0027: Excluir um Volume específico do banco de dados

**História de usuário:** Como Administrador, quero excluir um Volume cadastrado por engano, para manter a Edição correta.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Excluir um Volume específico do banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador" em "Gerenciar volumes"

Scenario: Caminho Feliz - Exclusão de Volume privado
Given que o Volume é "Privado"
When eu excluo o Volume e confirmo
Then o Volume deve deixar de existir

Scenario: Caminho Feliz - Exclusão do Volume 1 remove a capa da Edição (RN0069)
Given que a Edição é "Privado" e o Volume 1 é a origem da sua capa
When eu excluo o Volume 1 e confirmo
Then a Edição deve passar a aparecer sem capa e com um Volume a menos

Scenario: Caminho Alternativo 1 - Volume público (RN0041)
Given que o Volume é "Público"
When eu tento excluí-lo
Then a exclusão deve ser recusada e o Volume deve permanecer
# Divergência conhecida: ver SERS 6.3 (RN0041).
```

## RF0028: Consultar coleção de Volumes vinculados a uma Edição no banco de dados

**História de usuário:** Como Administrador, quero ver os Volumes de uma Edição em ordem, para gerenciar cada livro físico da publicação.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar coleção de Volumes vinculados a uma Edição no banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"

Scenario: Caminho Feliz - "Gerenciar volumes"
When eu abro "Gerenciar volumes" da 2ª edição da Obra "Obra Exemplo"
Then o caminho de navegação deve identificar a Obra "Obra Exemplo" e a 2ª edição
And os Volumes devem aparecer em ordem crescente de número, paginados
And cada Volume deve mostrar capa, número ou "Volume único", data conforme a precisão e visibilidade

Scenario: Caminho Alternativo 1 - Edição sem Volumes
When eu abro "Gerenciar volumes" de uma Edição sem Volumes
Then deve ser exibido um estado vazio, sem erro

Scenario: Caminho Alternativo 2 - Retorno preserva a página
Given que eu estou na página 2 de "Gerenciar volumes"
When eu abro o formulário de um Volume e volto
Then devo retornar à página 2
```

## RF0029: Consultar dados de um Volume específico do banco de dados

**História de usuário:** Como Administrador, quero ver todos os dados de um Volume, para auditar o registro no seu estado mais recente.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar dados de um Volume específico do banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"

Scenario: Caminho Feliz - Consulta do Volume
When eu abro os detalhes administrativos de um Volume existente
Then devo ver todos os seus dados no estado mais recente

Scenario: Caminho Alternativo 1 - Volume inexistente
When eu consulto um Volume que não existe
Then deve ser informado "Volume não encontrado."

Scenario: Caminho Alternativo 2 - Identificador inválido
When eu consulto um Volume com o identificador "abc"
Then deve ser informado "Formato de identificador invalido."
```

## RF0030: Cadastrar novo valor em uma lista de valores pré-cadastrados

**História de usuário:** Como Administrador, quero incluir valores nas listas administrativas, para mantê-las atualizadas para os formulários de Obra e de Edição.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Cadastrar novo valor em uma lista de valores pré-cadastrados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador" em "Gerenciar opções"

Scenario: Caminho Feliz - Inclusão de um valor com país relacionado
When eu escolho o formulário "Obra" e a lista "Autor"
And informo "Autor Exemplo" com o país "Japão"
Then o valor deve ser incluído e uma notificação de sucesso deve ser exibida

Scenario: Caminho Feliz - Inclusão de vários valores separados por vírgula (RN0043)
When eu escolho a lista "Editora brasileira" e informo "Editora A, Editora B"
Then os dois valores devem ser incluídos numa única operação

Scenario: Caminho Feliz - Vírgula faz parte do valor em "Formato"
When eu escolho a lista "Formato" e informo "13,5 x 20,5 cm"
Then um único valor "13,5 x 20,5 cm" deve ser incluído

Scenario: Caminho Alternativo 1 - Valor já existente (RN0043)
Given que a lista "Acabamento" já tem "Brochura"
When eu informo "brochura"
Then a inclusão deve ser recusada com "Essa lista já tem esse valor cadastrado: Brochura."

Scenario: Caminho Alternativo 2 - Valores repetidos no mesmo pedido (RN0043)
When eu informo "Editora C, editora c"
Then nenhuma inclusão deve ocorrer
And deve ser exibido "Valores repetidos na solicitação: Editora C."

Scenario: Caminho Alternativo 3 - País relacionado ausente
When eu informo um novo valor na lista "Pré-publicação" sem escolher país
Then a inclusão deve ser recusada com "Selecione ao menos um país de origem relacionado."

Scenario: Caminho Alternativo 4 - Valor vazio (RN0043)
When eu tento incluir um valor em branco
Then o campo deve exibir "Informe o texto do novo valor.", sem consultar a API e sem incluir nada
And uma inclusão em branco enviada diretamente à API deve ser recusada com "Categoria e texto do valor são obrigatórios."
When eu informo apenas separadores, como ", ,", numa lista que separa os valores por vírgula
Then deve ser exibido "Informe ao menos um valor válido.", sem incluir nada

Scenario: Caminho Alternativo 5 - Tipos de Obra e gêneros não são gerenciáveis (RN0066)
When eu consulto as listas disponíveis em "Gerenciar opções"
Then "Tipo de Obra" e "Gênero" não devem ser oferecidos
And uma inclusão enviada diretamente a essas listas deve ser recusada com "Os valores dessa lista são controlados pelo sistema e não podem ser criados."

@cancelado
Scenario: Gestão de tipos de Obra e gêneros
# Cancelado: tipos de Obra e gêneros são valores fixos do sistema, sem criação, renomeação, exclusão, ativação ou desativação pelo Administrador.
```

## RF0031: Alterar um valor específico de uma lista de valores pré-cadastrados

**História de usuário:** Como Administrador, quero corrigir o texto de um valor, para ajustar nomenclaturas sem quebrar os registros que já o usam.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Alterar um valor específico de uma lista de valores pré-cadastrados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador" em "Gerenciar opções"

Scenario: Caminho Feliz - Edição na própria linha
Given que a lista "Editora brasileira" tem "Editora Exmplo"
When eu edito o valor na própria linha para "Editora Exemplo" e confirmo
Then o valor deve ser atualizado
And as Edições que o usavam devem passar a exibir "Editora Exemplo"

Scenario: Caminho Alternativo 1 - Texto de outro valor (RN0043)
Given que a lista também tem "Editora Outra"
When eu renomeio "Editora Exemplo" para "Editora Outra"
Then a alteração deve ser recusada com "Essa lista já tem esse valor cadastrado!"

Scenario: Caminho Alternativo 2 - Texto em branco (RN0043)
When eu apago o texto e confirmo
Then a alteração deve ser recusada com "Informe o texto do novo valor.", sem consultar a API
And o valor anterior deve ser mantido
And uma renomeação em branco enviada diretamente à API deve ser recusada com "Texto do novo valor é obrigatório."

Scenario: Caminho Alternativo 3 - Valor fixo do sistema (RN0066)
When uma renomeação de tipo de Obra ou gênero é enviada diretamente
Then ela deve ser recusada com "Esse valor é controlado pelo sistema: só é possível ativá-lo ou desativá-lo."
```

## RF0032: Excluir um valor específico de uma lista de valores pré-cadastrados

**História de usuário:** Como Administrador, quero excluir valores obsoletos, para manter as listas limpas sem afetar registros existentes.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Excluir um valor específico de uma lista de valores pré-cadastrados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador" em "Gerenciar opções"

Scenario: Caminho Feliz - Exclusão de valor sem vínculos
Given que o valor não está vinculado a nenhuma Obra ou Edição
When eu excluo o valor e confirmo
Then o valor deve deixar de aparecer na lista

Scenario: Caminho Alternativo 1 - Valor em uso (RN0044)
Given que o valor está vinculado a ao menos uma Obra ou Edição
When eu tento excluí-lo
Then a exclusão deve ser recusada com "Esse valor está vinculado a um mangá, não pode ser excluído!"

Scenario: Caminho Alternativo 2 - Valor fixo do sistema (RN0066)
When a exclusão de um tipo de Obra ou gênero é enviada diretamente
Then ela deve ser recusada com "Esse valor é controlado pelo sistema e não pode ser excluído."
```

## RF0033: Consultar coleção de valores de listas pré-cadastradas

**História de usuário:** Como Administrador, quero consultar os valores de cada lista, para conferir as opções oferecidas nos formulários.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar coleção de valores de listas pré-cadastradas

Background:
Given que eu estou autenticado com o perfil ativo "Administrador" em "Gerenciar opções"

Scenario: Caminho Feliz - Listas agrupadas por formulário
When eu escolho o formulário "Edição"
Then devo ver as listas "Editora brasileira", "Acabamento", "Formato" e "Miolo"
When eu escolho o formulário "Obra"
Then devo ver as listas "Autor", "Pré-publicação" e "Editora original"

Scenario: Caminho Feliz - Busca e paginação
When eu escolho a lista "Autor" e pesquiso "Exemplo"
Then devo ver apenas os autores cujo nome contém "Exemplo", 5 por página, com seus países relacionados

Scenario: Caminho Alternativo 1 - Lista sem valores
When eu escolho uma lista sem valores
Then deve ser exibido um estado vazio, sem erro

Scenario: Caminho Alternativo 2 - Categoria inexistente
When uma lista inexistente é consultada diretamente
Then deve ser informado "Categoria não encontrada."

Scenario: Caminho Alternativo 3 - Formulário escolhido é lembrado
Given que eu escolhi o formulário "Edição"
When eu saio de "Gerenciar opções" e volto na mesma sessão
Then o formulário "Edição" deve continuar selecionado
```

## RF0034: Consultar coleção de contas de usuário cadastradas no banco de dados

**História de usuário:** Como Administrador, quero listar e filtrar as contas, para auditar perfis e localizar contas específicas.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar coleção de contas de usuário cadastradas no banco de dados

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"

Scenario: Caminho Feliz - Listagem de contas
When eu abro a gestão de usuários
Then devo ver as contas em ordem crescente de nome de usuário, 8 por página
And cada conta deve mostrar nome de usuário, e-mail, nível de acesso e status
And a linha da minha própria conta não deve permitir alteração

Scenario: Caminho Alternativo 1 - Busca e filtros combinados
When eu busco "dev" e filtro pelo nível "Usuário Padrão" e pelo status "Pendente"
Then devo ver apenas contas cujo nome ou e-mail contém "dev", sem o perfil Administrador e com o status "Pendente"

Scenario: Caminho Alternativo 2 - Ordenação invertida
When eu inverto a ordenação por usuário
Then as contas devem aparecer em ordem decrescente de nome de usuário

Scenario: Caminho Alternativo 3 - Nenhum resultado
When eu aplico filtros sem correspondência
Then deve ser exibido um estado vazio, sem erro
```

## RF0035: Conceder ou remover perfil Administrador

**História de usuário:** Como Administrador, quero conceder ou remover o perfil Administrador de outra conta, para controlar quem cuida do catálogo sem deixar o sistema sem administração.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Conceder ou remover perfil Administrador

Background:
Given que eu estou autenticado com o perfil ativo "Administrador" na gestão de usuários

Scenario: Caminho Feliz - Concessão do perfil Administrador
Given que a conta "leitor_exemplo" possui apenas "Usuário Padrão"
When eu altero o nível dessa conta para "Administrador"
Then a conta deve passar a possuir "Administrador" e "Usuário Padrão"
And no próximo login dela, a sessão deve começar com o perfil ativo "Administrador"

Scenario: Caminho Feliz - Remoção do perfil Administrador (RN0062)
Given que a conta "curador_exemplo" possui "Administrador" e está com uma sessão aberta nesse perfil
And existe outro Administrador ativo
When eu altero o nível dessa conta para "Usuário Padrão"
Then a conta deve manter apenas "Usuário Padrão"
And a sessão aberta dela deve perder o acesso às páginas administrativas

Scenario: Caminho Alternativo 1 - Própria conta (RN0045)
When eu tento alterar o nível da minha própria conta
Then a alteração deve ser recusada com "Você não pode alterar o nível de acesso de sua própria conta!"

Scenario: Caminho Alternativo 2 - Remoções simultâneas não eliminam o último Administrador (RN0063)
Given que eu e "curador_exemplo" somos os dois únicos Administradores ativos
When eu removo o perfil Administrador de "curador_exemplo" ao mesmo tempo em que ele remove o meu
Then apenas uma das remoções deve ser aceita
And a outra deve ser recusada com "Operação bloqueada: o sistema ficaria sem nenhum administrador ativo."

Scenario: Caminho Alternativo 3 - Falha na alteração
When a alteração é recusada pela API
Then a tabela deve restaurar o nível anterior e exibir a mensagem recebida
```

## RF0036: Consultar vitrine pública de Obras

**História de usuário:** Como visitante ou Usuário Padrão, quero buscar e filtrar as Obras publicadas, para encontrar títulos do meu interesse.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar vitrine pública de Obras

Scenario: Caminho Feliz - Vitrine padrão
Given que existem Obras públicas
When eu abro "/pesquisa"
Then devo ver a aba de Obras com as Obras públicas em ordem de título, paginadas, com capas em proporção 2:3
And a aba, a ordenação e a página devem ficar registradas na URL

Scenario: Caminho Alternativo 1 - Busca por título, título romanizado ou autor
When eu busco "Ekusanpuru"
Then devo ver a Obra cujo título romanizado contém "Ekusanpuru"
When eu busco pelo nome de um autor
Then devo ver as Obras públicas em que ele tem crédito

Scenario: Caminho Alternativo 2 - Filtros combinados por interseção (RN0047)
When eu seleciono os gêneros "Ação" e "Comédia" e a demografia "Shonen"
Then devo ver somente Obras que tenham "Ação", "Comédia" e "Shonen" ao mesmo tempo
And os filtros devem ficar registrados na URL

Scenario: Caminho Alternativo 3 - Tipos de Obra restritos ao país
When eu seleciono o país "China"
Then os tipos oferecidos devem ser apenas os compatíveis com a China
And um tipo incompatível já selecionado deve ser limpo da URL e da consulta

Scenario: Caminho Alternativo 4 - Registros privados não aparecem (RN0046)
Given que uma Obra é "Privado"
When eu busco pelo título dela
Then ela não deve aparecer

Scenario Outline: Caminho Alternativo 5 - Conteúdo adulto conforme quem consulta (RN0048)
Given que existe uma Obra pública marcada como conteúdo adulto
And eu sou <quem consulta>
When eu consulto a vitrine de Obras
Then a Obra adulta <resultado>

Examples:
  | quem consulta                                                              | resultado            |
  | visitante                                                                  | não deve aparecer    |
  | Usuário Padrão menor de 18 anos                                            | não deve aparecer    |
  | Usuário Padrão maior de idade com a preferência desativada                 | não deve aparecer    |
  | Usuário Padrão maior de idade com a preferência ativada                    | deve aparecer        |
  | conta com o perfil Administrador concedido, navegando como Usuário Padrão, menor de idade e sem preferência | deve aparecer |

Scenario: Caminho Alternativo 6 - Gênero restrito fora dos filtros (RN0048)
Given que eu sou visitante
When eu abro os filtros de gênero
Then o gênero "Hentai" não deve ser oferecido

Scenario: Caminho Alternativo 7 - Obra com Hentai sem indicação adulta também é ocultada (RN0048)
Given que uma Obra pública tem o gênero "Hentai" mas, por dado legado, não está marcada como adulta
When um visitante consulta a vitrine
Then essa Obra não deve aparecer

Scenario: Caminho Alternativo 8 - Nenhum resultado (RNF16)
When eu aplico filtros sem correspondência
Then deve ser exibido um estado vazio com a opção de limpar os filtros
```

## RF0037: Consultar vitrine pública de Edições

**História de usuário:** Como visitante ou Usuário Padrão, quero buscar as Edições brasileiras, para encontrar publicações por editora e características físicas.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar vitrine pública de Edições

Scenario: Caminho Feliz - Busca de Edições pela Obra
Given que existem Edições públicas de Obras públicas
When eu abro a aba de Edições e busco pelo título, título original ou autor de uma Obra
Then devo ver as Edições públicas dessa Obra, paginadas
And cada Edição deve mostrar a capa do seu Volume 1 público, o título da Obra e "Nª edição · editora"

Scenario: Caminho Alternativo 1 - Filtros próprios da Edição
When eu filtro por editora brasileira, formato, acabamento, número da edição e status no Brasil
Then devo ver somente as Edições que atendem a todos os filtros

Scenario: Caminho Alternativo 2 - Preservação do contexto entre abas (RN0049)
Given que eu busquei "Obra Exemplo" na aba de Obras
When eu alterno para a aba de Edições
Then o termo "Obra Exemplo" e a ordenação compatível devem ser mantidos

Scenario: Caminho Alternativo 3 - Edições privadas ou de Obras privadas não aparecem (RN0046)
Given que uma Edição é "Privado" ou pertence a uma Obra "Privado"
When eu busco por ela
Then ela não deve aparecer
```

## RF0038: Consultar detalhes públicos de uma Obra

**História de usuário:** Como visitante ou Usuário Padrão, quero ver a ficha completa de uma Obra, para conhecer sua autoria, classificação e publicações no Brasil.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar detalhes públicos de uma Obra

Scenario: Caminho Feliz - Ficha pública pela URL da Obra (RN0071)
Given que a Obra "Obra Exemplo" é "Público"
When eu abro "/obras/obra-exemplo"
Then devo ver títulos, capa 2:3, país, tipo, autores com papéis, editoras originais, pré-publicações, demografias, gêneros, status e anos da publicação original, sinopse e Edições públicas

Scenario: Caminho Feliz - Ordem dos créditos de autoria (RN0023)
Given que a Obra tem os autores "Beatriz" (Arte), "Ana" (Arte) e "Carlos" (Criador Original)
When eu abro a ficha da Obra
Then os autores devem aparecer na ordem "Carlos", "Ana" e "Beatriz"

Scenario: Caminho Alternativo 1 - Metadados ausentes não são inventados (RN0072)
Given que a Obra não tem título original nem pré-publicação
When eu abro a ficha
Then esses itens devem ser omitidos, sem valores genéricos
# Divergência conhecida: ver SERS 6.3 (RN0072).

Scenario: Caminho Alternativo 2 - Obra privada, adulta ou inexistente (RN0046, RN0048)
Given que a Obra é "Privado", ou é adulta e eu não posso ver conteúdo adulto
When eu abro a URL da Obra
Then deve ser exibido um erro genérico de Obra não encontrada, sem revelar o motivo
And deve haver a opção de tentar novamente

Scenario: Caminho Alternativo 3 - Capa indisponível (RNF15)
Given que a capa da Obra não pode ser carregada
When eu abro a ficha
Then deve ser exibido o estado "Sem capa" na proporção 2:3

@futuro
Scenario: Enriquecimento para usuário autenticado (RN0050)
Given que eu possuo uma sessão válida
When eu abro a ficha de uma Obra pública
Then a resposta poderá incluir meus dados de posse e desejo

@cancelado
Scenario: Número de volumes originais e indicador de conteúdo adulto na ficha
# Cancelado: a ficha não exibe esses dados.
```

## RF0039: Consultar Edições públicas vinculadas a uma Obra

**História de usuário:** Como visitante ou Usuário Padrão, quero comparar as Edições brasileiras de uma Obra, para escolher qual acompanhar.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar Edições públicas vinculadas a uma Obra

Scenario: Caminho Feliz - Prévia das Edições na ficha da Obra (RN0051)
Given que a Obra pública tem Edições públicas
When eu abro a ficha da Obra
Then cada Edição deve mostrar número, editora e status acompanhado do total de Volumes públicos
And deve exibir a prévia dos três primeiros Volumes públicos

Scenario: Caminho Alternativo 1 - Dependências privadas omitidas (RN0046)
Given que a Obra tem uma Edição privada e uma Edição pública com um Volume privado
When eu abro a ficha da Obra
Then somente a Edição pública deve aparecer, sem o Volume privado

Scenario: Caminho Alternativo 2 - Obra sem Edições públicas
When eu abro a ficha de uma Obra sem Edições públicas
Then deve ser exibido um estado vazio
```

## RF0040: Consultar detalhes públicos de uma Edição

**História de usuário:** Como visitante ou Usuário Padrão, quero ver uma Edição e seus Volumes, para acompanhar a publicação brasileira.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar detalhes públicos de uma Edição

Scenario: Caminho Feliz - Ficha da Edição pela URL contextualizada (RN0071)
Given que a 2ª edição (identificador 20) da Obra "obra-exemplo" é pública
When eu abro "/obras/obra-exemplo/edicao/20"
Then devo ver o título da Obra, a capa derivada, a editora, o número, o status, o total de Volumes públicos e a lista paginada de Volumes
And o link da Obra deve levar a "/obras/obra-exemplo"
And cada Volume deve levar a "/obras/obra-exemplo/edicao/20/volume/{id do Volume}"

Scenario: Caminho Feliz - Vários miolos na Edição
Given que a Edição tem os miolos "Offset" e "Couché", nessa ordem
When eu abro a ficha da Edição
Then o campo "Miolo" deve exibir "Offset" e "Couché" empilhados, nessa ordem

Scenario: Caminho Feliz - Anos de publicação no Brasil
Given que a Edição é "Completa" e os Volumes públicos vão de 2020 a 2024
When eu abro a ficha da Edição
Then o início deve ser 2020 e o fim 2024

Scenario: Caminho Alternativo 1 - Contexto divergente na URL (RN0071)
Given que a Edição 20 pertence à Obra "obra-exemplo"
When eu abro "/obras/outra-obra/edicao/20"
Then deve ser exibido "Edição não encontrada."

Scenario: Caminho Alternativo 2 - Rota isolada não é suportada (RN0071)
When eu abro "/edicoes/20"
Then deve ser exibida a página de rota não encontrada, sem redirecionamento

Scenario: Caminho Alternativo 3 - Metadados ausentes (RN0072)
Given que a Edição não informa acabamento, formato nem miolo
When eu abro a ficha da Edição
Then esses itens devem ser omitidos
# Divergência conhecida: ver SERS 6.3 (RN0072 e RNF15).

Scenario: Caminho Alternativo 4 - Sem Volumes públicos
When eu abro uma Edição pública sem Volumes públicos
Then deve ser exibido um estado vazio

Scenario: Caminho Alternativo 5 - Paginação na URL
When eu avanço para a página 2 de Volumes
Then a página deve ficar registrada na URL
```

## RF0041: Consultar detalhes públicos de um Volume

**História de usuário:** Como visitante ou Usuário Padrão, quero ver os detalhes de um Volume e navegar entre os Volumes da mesma Edição, para consultar lançamentos, preços e identificadores.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar detalhes públicos de um Volume

Background:
Given que a 2ª edição (identificador 20) da Obra "obra-exemplo" é pública

Scenario: Caminho Feliz - Ficha do Volume pela URL contextualizada (RN0071)
Given que o Volume 1 (identificador 30) é público
When eu abro "/obras/obra-exemplo/edicao/20/volume/30"
Then devo ver a capa 2:3, a data "20/08/2026", "416 páginas", "R$ 79,90", ISBN-10, ISBN-13 e sinopse
And devo ver os links "Voltar para 2ª edição" e "Ver 2ª edição" para "/obras/obra-exemplo/edicao/20"
And devo ver o link da Obra para "/obras/obra-exemplo"

Scenario: Caminho Feliz - Navegação para os Volumes anterior e seguinte (RN0070)
Given que a Edição tem os Volumes públicos 0, 1 e 2
When eu abro o Volume 1
Then abaixo da capa devem aparecer "Volume 0" à esquerda e "Volume 2" à direita
And "Volume 0" deve levar a "/obras/obra-exemplo/edicao/20/volume/{id do Volume 0}"
And "Volume 2" deve levar a "/obras/obra-exemplo/edicao/20/volume/{id do Volume 2}"

Scenario: Caminho Alternativo 1 - Primeiro Volume só tem seguinte (RN0070)
Given que a Edição tem os Volumes públicos 1 e 2
When eu abro o Volume 1
Then apenas o botão "Volume 2" deve aparecer

Scenario: Caminho Alternativo 2 - Último Volume só tem anterior (RN0070)
Given que a Edição tem os Volumes públicos 1 e 2
When eu abro o Volume 2
Then apenas o botão "Volume 1" deve aparecer

Scenario: Caminho Alternativo 3 - Volume único não tem navegação (RN0070)
Given que a Edição tem um único Volume público
When eu abro esse Volume
Then nenhum botão de navegação entre Volumes deve aparecer

Scenario: Caminho Alternativo 4 - Volume privado é pulado (RN0070)
Given que a Edição tem os Volumes 1 e 3 públicos e o Volume 2 privado
When eu abro o Volume 1
Then o botão seguinte deve ser "Volume 3"
And nenhum botão deve levar ao Volume 2

Scenario: Caminho Alternativo 5 - Campos opcionais ausentes (RN0072)
Given que o Volume não tem páginas, preço, ISBN, sinopse nem link de afiliado
When eu abro o Volume
Then "Número de páginas", "Preço", "ISBN-10", "ISBN-13" e "Sinopse" não devem aparecer
And o campo "Miolo" não deve aparecer na ficha do Volume
And o botão "Comprar na Amazon" deve aparecer desabilitado com a marca "Indisponível"

Scenario: Caminho Alternativo 6 - Link de afiliado
Given que o Volume tem link de afiliado
When eu escolho "Comprar na Amazon"
Then a loja deve abrir em nova aba

Scenario: Caminho Alternativo 7 - Contexto divergente ou identificador inválido (RN0071)
When eu abro "/obras/outra-obra/edicao/20/volume/30" ou "/obras/obra-exemplo/edicao/20/volume/invalido"
Then deve ser exibido "Volume não encontrado." com as opções "Tentar novamente" e "Voltar ao catálogo"

Scenario: Caminho Alternativo 8 - Volume indisponível publicamente (RN0046)
Given que a 1ª edição (identificador 10) da Obra "obra-exemplo" é "Privado" e tem o Volume de identificador 15
And a Obra "obra-privada" é "Privado" e tem a Edição 40 com o Volume 50
When eu abro "/obras/obra-exemplo/edicao/10/volume/15" ou "/obras/obra-privada/edicao/40/volume/50"
Then deve ser exibido "Volume não encontrado." sem revelar qual nível é privado

Scenario: Caminho Alternativo 9 - Rota isolada não é suportada (RN0071)
When eu abro "/volumes/30"
Then deve ser exibida a página de rota não encontrada, sem redirecionamento
```

## RF0042: Consultar calendário público de lançamentos

**História de usuário:** Como visitante ou usuário, quero consultar os Volumes previstos para um mês, para acompanhar os lançamentos no Brasil.

### Critérios de Aceite (Gherkin):

```gherkin
@futuro
Feature: Consultar calendário público de lançamentos
# Futuro (planejado): hoje existe apenas a tela "Checklist" com "Em breve: Acompanhe seus lançamentos mensais."

Scenario: Caminho Feliz - Consulta por mês e ano
Given que existem Volumes públicos com lançamento no mês informado
When eu consulto o calendário informando mês e ano
Then devo ver os Volumes públicos previstos para esse período

Scenario: Caminho Alternativo 1 - Período padrão (RN0052)
When eu abro o calendário sem informar mês e ano
Then o calendário deve usar o mês e o ano vigentes
And Volumes com data apenas anual não devem ser atribuídos a um mês

Scenario: Caminho Alternativo 2 - Busca e filtro por editora
When eu busco por título ou autor e filtro por editora brasileira
Then devo ver somente os lançamentos que atendem aos critérios
```

## RF0043: Registrar posse individual de Volume

**História de usuário:** Como usuário autenticado, quero marcar um Volume como adquirido, para registrá-lo na minha Estante Digital.

### Critérios de Aceite (Gherkin):

```gherkin
@futuro
Feature: Registrar posse individual de Volume
# Futuro (planejado): o botão "Coleção" da página do Volume ainda não tem ação.

Scenario: Caminho Feliz - Adição à Estante Digital (RN0053, RN0054, RN0055)
Given que eu tenho uma sessão válida e o Volume é público
When eu marco o Volume como adquirido
Then ele deve aparecer na minha Estante Digital como "não lido"
And deve ser retirado da minha Lista de Desejos, se estiver nela

Scenario: Caminho Alternativo 1 - Volume indisponível (RN0056)
Given que o Volume, a Edição ou a Obra é privado
When eu tento adicioná-lo à Estante
Then a operação deve ser recusada
```

## RF0044: Atualizar estado de leitura ou remover posse individual de Volume

**História de usuário:** Como usuário autenticado, quero marcar Volumes como lidos e retirá-los da Estante, para manter minha coleção atualizada.

### Critérios de Aceite (Gherkin):

```gherkin
@futuro
Feature: Atualizar estado de leitura ou remover posse individual de Volume

Scenario: Caminho Feliz - Marcação como lido (RN0055)
Given que o Volume está na minha Estante Digital
When eu o marco como lido
Then ele deve aparecer como "lido" na minha Estante

Scenario: Caminho Feliz - Remoção da posse
Given que o Volume está na minha Estante Digital
When eu removo a posse
Then ele deve sair da minha Estante, junto com o estado de leitura
And deve continuar existindo no catálogo
```

## RF0045: Sincronizar registros de posse em lote

**História de usuário:** Como usuário autenticado, quero marcar e desmarcar vários Volumes de uma Edição de uma vez, para atualizar minha coleção com rapidez.

### Critérios de Aceite (Gherkin):

```gherkin
@futuro
Feature: Sincronizar registros de posse em lote
# Futuro (planejado): a seleção visual de Volumes já existe na Web, mas não grava nada.

Scenario: Caminho Feliz - Sincronização em lote
Given que eu tenho uma sessão válida e estou na seleção de Volumes de uma Edição pública
When eu confirmo uma seleção com Volumes a adicionar e a remover
Then toda a alteração deve ser aplicada de uma vez
And os Volumes adicionados devem sair da Lista de Desejos e começar como "não lidos"

Scenario: Caminho Alternativo 1 - Falha em algum item
Given que um item da seleção viola uma regra de posse ou visibilidade
When a sincronização é processada
Then nenhuma alteração deve ser aplicada
```

## RF0046: Consultar registros da Estante Digital

**História de usuário:** Como usuário autenticado, quero ver minha Estante Digital agrupada por Obra e Edição, para acompanhar minha coleção.

### Critérios de Aceite (Gherkin):

```gherkin
@futuro
Feature: Consultar registros da Estante Digital
# Futuro (planejado): hoje "/colecao" exibe apenas "Em breve: Gerencie seus volumes lidos e comprados."

Scenario: Caminho Feliz - Estante agrupada
Given que eu tenho Volumes públicos na Estante
When eu abro a Estante Digital
Then devo ver os Volumes agrupados por Obra e Edição, com a progressão e o estado de leitura

Scenario: Caminho Alternativo 1 - Volume que deixou de ser público (RN0056)
Given que um Volume da minha Estante passou a ser privado
When eu abro a Estante
Then ele deve ser omitido sem ser apagado
```

## RF0047: Registrar intenção de compra

**História de usuário:** Como usuário autenticado, quero adicionar um Volume à Lista de Desejos, para lembrar do que pretendo comprar.

### Critérios de Aceite (Gherkin):

```gherkin
@futuro
Feature: Registrar intenção de compra
# Futuro (planejado): o botão "Lista de Desejos" da página do Volume ainda não tem ação.

Scenario: Caminho Feliz - Adição à Lista de Desejos
Given que eu tenho uma sessão válida e não possuo o Volume na Estante
When eu adiciono o Volume público à Lista de Desejos
Then ele deve aparecer uma única vez na minha Lista de Desejos

Scenario: Caminho Alternativo 1 - Volume já possuído (RN0057)
Given que o Volume já está na minha Estante Digital
When eu tento adicioná-lo à Lista de Desejos
Then a operação deve ser recusada informando que o Volume já está na Estante
```

## RF0048: Remover intenção de compra

**História de usuário:** Como usuário autenticado, quero retirar um Volume da Lista de Desejos, para mantê-la atualizada.

### Critérios de Aceite (Gherkin):

```gherkin
@futuro
Feature: Remover intenção de compra

Scenario: Caminho Feliz - Remoção da Lista de Desejos
Given que o Volume está na minha Lista de Desejos
When eu o removo
Then ele deve sair da Lista de Desejos e continuar existindo no catálogo
```

## RF0049: Consultar Lista de Desejos

**História de usuário:** Como usuário autenticado, quero ver minha Lista de Desejos, para acompanhar o que ainda pretendo comprar.

### Critérios de Aceite (Gherkin):

```gherkin
@futuro
Feature: Consultar Lista de Desejos
# Futuro (planejado): hoje "/desejos" exibe apenas "Em breve: Mangás que você deseja adquirir."

Scenario: Caminho Feliz - Consulta da Lista de Desejos
Given que eu tenho Volumes públicos na Lista de Desejos
When eu abro a Lista de Desejos
Then devo ver esses Volumes com seus dados públicos essenciais

Scenario: Caminho Alternativo 1 - Item que se tornou privado (RN0056)
Given que um Volume desejado passou a ser privado
When eu abro a Lista de Desejos
Then ele deve ser omitido enquanto estiver indisponível
```

## RF0050: Importar e gerenciar capa interna por URL

**História de usuário:** Como Administrador, quero informar a URL de uma imagem para que o CoMangá a importe e guarde, para gerenciar capas sem baixar arquivos manualmente.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Importar e gerenciar capa interna por URL

Background:
Given que eu estou autenticado com o perfil ativo "Administrador"

Scenario: Caminho Feliz - Importação de capa no formulário de Obra ou Volume (RN0058)
When eu informo a URL HTTPS pública de uma imagem JPEG, PNG, WebP ou AVIF e importo
Then a prévia da capa importada deve ser exibida
And ao salvar o formulário, a Obra ou o Volume deve passar a exibir essa capa pelo endereço do CoMangá, não pela URL informada

Scenario: Caminho Alternativo 1 - URL inválida ou de rede privada (RN0059)
When eu informo "http://exemplo.com/capa.jpg" ou uma URL que aponta para "localhost" ou rede privada
Then a importação deve ser recusada com "Informe uma URL HTTPS pública válida para a capa."
And nenhuma capa deve ser associada

Scenario: Caminho Alternativo 2 - Imagem acima do limite (RN0059)
When eu informo a URL de uma imagem maior que o limite configurado
Then a importação deve ser recusada com "A imagem excede o tamanho máximo permitido."

Scenario: Caminho Alternativo 3 - Falha na substituição preserva a capa atual (RN0060)
Given que o Volume já tem uma capa
When a importação ou a associação da nova capa falha
Then o Volume deve continuar exibindo a capa anterior

Scenario: Caminho Alternativo 4 - Capa pendente descartada
Given que eu importei uma capa e ainda não salvei o formulário
When eu descarto essa capa
Then ela deve deixar de existir e não deve ser associada a nenhum registro

Scenario: Caminho Alternativo 5 - Capa associada não pode ser descartada (RN0060)
When o descarte de uma capa já associada é solicitado
Then ele deve ser recusado com "A capa já está associada a um registro."

Scenario: Caminho Alternativo 6 - Não há remoção de capa sem substituta (RN0061)
Given que a Obra tem capa
When eu salvo a Obra sem informar nova capa
Then a capa atual deve ser mantida

Scenario: Caminho Alternativo 7 - Edição não importa capa (RN0069)
When eu abro o formulário de Edição
Then não deve haver importação de capa, pois a capa da Edição vem do Volume 1
```

## RF0051: Consultar Obras públicas de um Autor

**História de usuário:** Como visitante ou Usuário Padrão, quero ver as Obras de um autor, para conhecer outros trabalhos dele publicados no Brasil.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Consultar Obras públicas de um Autor

Scenario: Caminho Feliz - Página do Autor
Given que o autor 7 tem crédito em Obras públicas
When eu abro "/autores/7"
Then devo ver o nome do autor e suas Obras públicas na mesma grade da vitrine, paginadas
And a página deve ficar registrada na URL

Scenario: Caminho Alternativo 1 - Conteúdo adulto e registros privados (RN0046, RN0048)
Given que o autor tem uma Obra privada e uma Obra adulta
When um visitante abre a página do Autor
Then nenhuma das duas deve aparecer

Scenario: Caminho Alternativo 2 - Autor sem Obras públicas
When eu abro a página de um autor sem Obras públicas
Then deve ser exibido um estado vazio com retorno ao catálogo

Scenario: Caminho Alternativo 3 - Identificador inválido ou autor inexistente
When eu abro "/autores/abc" ou a página de um autor inexistente
Then deve ser exibido "Autor não encontrado." com a opção "Tentar novamente"
```

## RF0052: Alternar perfil ativo da sessão

**História de usuário:** Como dono de uma conta com os perfis Administrador e Usuário Padrão, quero escolher qual perfil usar, para alternar entre a curadoria e a navegação pelo catálogo.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Alternar perfil ativo da sessão

Scenario: Caminho Feliz - Troca para Usuário Padrão
Given que eu possuo os perfis "Administrador" e "Usuário Padrão"
And o meu perfil ativo é "Administrador"
When eu escolho "Usuário Padrão" no seletor "Perfil ativo" da página de perfil
Then o sistema deve exibir "Perfil ativo alterado para Usuário Padrão."
And sem recarregar a página, eu devo poder abrir o catálogo público
And as páginas administrativas devem deixar de estar disponíveis nesta sessão

Scenario: Caminho Feliz - A troca não altera os perfis da conta (RN0062)
Given que eu troquei o perfil ativo para "Usuário Padrão"
When eu consulto meus perfis
Then a conta deve continuar possuindo "Administrador" e "Usuário Padrão"

Scenario: Caminho Feliz - A troca vale para esta sessão e para o próximo login (RN0062)
Given que eu tenho duas sessões abertas com o perfil ativo "Administrador"
When eu troco o perfil ativo para "Usuário Padrão" na primeira sessão
Then a segunda sessão deve continuar com "Administrador"
And no próximo login, a sessão deve começar com "Usuário Padrão"

Scenario: Caminho Alternativo 1 - Conta só com Usuário Padrão
Given que a minha conta possui apenas "Usuário Padrão"
When eu abro a página de perfil
Then o seletor de perfil não deve ser exibido
And uma tentativa enviada diretamente para ativar "Administrador" deve ser recusada com "Acesso negado: sua conta não possui este perfil de acesso."

Scenario: Caminho Alternativo 2 - Perfil inexistente
When uma troca para o perfil "Moderador" é enviada diretamente
Then ela deve ser recusada com "Perfil de acesso inexistente." e o perfil ativo deve ser mantido

Scenario: Caminho Alternativo 3 - Recusa preserva o perfil anterior
When a API recusa a troca de perfil
Then o seletor deve voltar ao perfil anterior e a mensagem de erro deve ser exibida
```

## RF0053: Direcionar o acesso às páginas conforme sessão e perfil ativo

**História de usuário:** Como pessoa que usa o CoMangá, quero ser levada à página adequada à minha sessão e ao meu perfil ativo, para não cair em telas que não se aplicam a mim nem acessar áreas sem permissão.

### Critérios de Aceite (Gherkin):

```gherkin
@vigente
Feature: Direcionar o acesso às páginas conforme sessão e perfil ativo

Scenario Outline: Páginas de visitante levam o usuário autenticado ao perfil (RN0065)
Given que eu estou autenticado como "leitor_exemplo"
When eu abro "<pagina>"
Then eu devo ser levado a "/perfil/leitor_exemplo"

Examples:
  | pagina                    |
  | /entrar                   |
  | /cadastrar                |
  | /reenvio                  |
  | /recuperar-senha          |
  | /redefinir-senha/abc123   |

Scenario: Visitante na raiz é levado ao login (RN0065)
Given que eu sou visitante
When eu abro "/"
Then eu devo ser levado a "/entrar"

Scenario: Visitante em página protegida é levado ao login (RN0014, RN0065)
Given que eu sou visitante
When eu abro "/perfil" ou "/admin/gerenciar-mangas"
Then eu devo ser levado a "/entrar"
And deve ser exibido "Sua sessão é inválida ou foi encerrada. Por favor, faça login novamente."

Scenario: Usuário Padrão não acessa páginas administrativas (RN0064)
Given que eu estou autenticado como "leitor_exemplo" com o perfil ativo "Usuário Padrão"
When eu abro "/admin/opcoes"
Then eu devo ser levado a "/perfil/leitor_exemplo"
And deve ser exibido "Acesso negado: Você não tem permissão para acessar esta área."

Scenario: Administrador com Usuário Padrão ativo também não acessa páginas administrativas (RN0064)
Given que eu possuo o perfil "Administrador", mas o meu perfil ativo é "Usuário Padrão"
When eu abro "/admin/gerenciar-mangas"
Then eu devo ser levado ao meu perfil com o aviso "Acesso negado: Você não tem permissão para acessar esta área."

Scenario: A API recusa operação administrativa sem o perfil Administrador ativo (RN0064, RNF07)
Given que a sessão tem o perfil ativo "Usuário Padrão"
When uma operação administrativa é enviada diretamente à API
Then ela deve ser recusada com "Acesso negado: voce nao possui permissao para executar esta operacao."
And nenhum dado deve ser alterado

Scenario: Administrador ativo acessa as páginas administrativas (RN0064)
Given que o meu perfil ativo é "Administrador"
When eu abro "/admin/gerenciar-mangas"
Then devo ver "Gerenciar mangás"

Scenario Outline: Administrador ativo não acessa páginas públicas do catálogo (RN0065)
Given que eu estou autenticado como "curador_exemplo" com o perfil ativo "Administrador"
When eu abro "<pagina>"
Then eu devo ser levado a "/perfil/curador_exemplo"
And deve ser exibido o aviso "Mude o perfil para usuário padrão para acessar essa página."

Examples:
  | pagina                                      |
  | /pesquisa                                   |
  | /autores/7                                  |
  | /obras/obra-exemplo                         |
  | /obras/obra-exemplo/edicao/20               |
  | /obras/obra-exemplo/edicao/20/volume/30     |
  | /obras/obra-exemplo/edicao/20/selecionar/estante |

Scenario: Após trocar para Usuário Padrão, o catálogo público fica acessível (RN0065)
Given que eu troquei o perfil ativo de "Administrador" para "Usuário Padrão"
When eu abro "/obras/obra-exemplo"
Then devo ver a ficha pública da Obra

Scenario: Visitante e Usuário Padrão acessam as páginas públicas (RN0065)
Given que eu sou visitante ou estou com o perfil ativo "Usuário Padrão"
When eu abro "/pesquisa"
Then devo ver a vitrine pública

Scenario: Link de ativação não depende da sessão (RN0065)
When qualquer pessoa abre "/activate/{token}"
Then a tela de ativação deve ser exibida sem redirecionamento
```

## RF0054: Bloquear e desbloquear conta de usuário

**História de usuário:** Como Administrador, quero bloquear e desbloquear outra conta, para impedir o acesso de quem viola as regras do catálogo.

### Critérios de Aceite (Gherkin):

```gherkin
@futuro
Feature: Bloquear e desbloquear conta de usuário
# Futuro (planejado): o status "Bloqueada" já impede login, reenvio e recuperação, mas ainda não há fluxo que o atribua.

Scenario: Caminho Feliz - Conta bloqueada não consegue entrar (RN0013)
Given que o Administrador bloqueou a conta "leitor_exemplo"
When "leitor_exemplo" tenta entrar com credenciais válidas
Then o acesso deve ser recusado
And deve ser exibido "Esta conta foi bloqueada por razões de segurança."

Scenario: Caminho Alternativo 1 - Conta desbloqueada volta a entrar
Given que o Administrador desbloqueou a conta "leitor_exemplo"
When "leitor_exemplo" entra com credenciais válidas
Then a sessão deve ser criada normalmente
```
