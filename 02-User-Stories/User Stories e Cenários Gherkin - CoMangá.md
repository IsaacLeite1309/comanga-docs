# User Stories e Cenários Gherkin do CoMangá

## RF0001: Cadastrar conta de acesso e disparar e-mail com link de ativação

**História de usuário:** Como um usuário visitante, Quero cadastrar uma nova conta de acesso informando meus dados pessoais básicos, Para que eu possa receber o link de ativação, validar minha identidade e acessar as funcionalidades restritas do sistema.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Cadastrar conta de acesso e disparar e-mail com link de ativação

Scenario: Caminho Feliz - Sucesso no cadastro de nova conta e envio de e-mail de ativação
Given que eu sou um usuário visitante na tela de registro
And o e-mail "novo@usuario.com" e o nome de usuário "novo_user" não existem no banco de dados
When eu preencho o campo de Nome de Usuário com "novo_user"
And eu preencho o campo de Data de Nascimento com "1990-08-15"
And eu preencho o campo de E-mail com "novo@usuario.com"
And eu preencho os campos de Senha e Confirmação de Senha com "SenhaForte123!"
And submeto o formulário de cadastro
Then o sistema deve gravar os dados da nova conta
And deve atribuir o status inicial da conta como "Pendente"
And deve atribuir compulsoriamente o nível de acesso como "Usuário Padrão"
And deve atribuir compulsoriamente a preferência de conteúdo adulto (+18) como "Desativada"
And deve gerar um token de ativação para a conta
And deve disparar automaticamente um e-mail para "novo@usuario.com" contendo o link de ativação associado ao token

Scenario: Caminho Alternativo 1 - Falha no cadastro por violação da formatação do Nome de Usuário (RN0001)
Given que eu sou um usuário visitante na tela de registro
When eu preencho o campo de Nome de Usuário com "User Inválido!"
And preencho os demais campos corretamente
And submeto o formulário de cadastro
Then o sistema deve recusar a transação
And deve exibir a mensagem de erro no campo de usuário: "Utilize entre 3 e 20 caracteres, sem espaços, acentos ou caracteres especiais."

Scenario: Caminho Alternativo 2 - Falha no cadastro por senha fora do padrão de força estrutural (RN0002)
Given que eu sou um usuário visitante na tela de registro
When eu preencho o campo de Senha e Confirmação de Senha com "fraca12"
And preencho os demais campos corretamente
And submeto o formulário de cadastro
Then o sistema deve recusar a transação
And deve exibir a mensagem de erro no campo de senha: "Utilize no mínimo 8 caracteres, incluindo pelo menos uma letra maiúscula, uma minúscula, um número e um caractere especial."

Scenario: Caminho Alternativo 3 - Falha no cadastro por duplicidade de e-mail (RN0003)
Given que eu sou um usuário visitante na tela de registro
And o e-mail "existente@usuario.com" já está vinculado a uma conta no banco de dados
When eu preencho o campo de E-mail com "existente@usuario.com"
And preencho os demais campos corretamente
And submeto o formulário de cadastro
Then o sistema deve recusar a transação
And deve exibir a mensagem de erro no campo de e-mail: "Este endereço de e-mail já está em uso. Tente fazer login ou recuperar sua senha."

Scenario: Caminho Alternativo 4 - Falha no cadastro por colisão de nome de usuário (RN0004)
Given que eu sou um usuário visitante na tela de registro
And o nome de usuário "user_existente" já está cadastrado no banco de dados
When eu preencho o campo de Nome de Usuário com "user_existente"
And preencho os demais campos corretamente
And submeto o formulário de cadastro
Then o sistema deve recusar a transação
And deve exibir a mensagem de erro no campo de usuário: "Este nome de usuário não está disponível. Por favor, escolha outro."

Scenario: Caminho Alternativo 5 - Falha no cadastro por divergência na confirmação de senha (RN0020)
Given que eu sou um usuário visitante na tela de registro
When eu preencho o campo de Senha com "SenhaForte123!"
And preencho o campo de Confirmação de Senha com "SenhaDiferente123!"
And preencho os demais campos corretamente
And submeto o formulário de cadastro
Then o sistema deve recusar a transação
And deve exibir a mensagem de erro: "Divergência nos valores da senha e confirmação de senha!"

Scenario: Caminho Alternativo 6 - Falha no cadastro por data de nascimento inválida ou futura (RN0048)
Given que eu sou um usuário visitante na tela de registro
When eu preencho o campo de Data de Nascimento com "2099-01-01"
And preencho os demais campos corretamente
And submeto o formulário de cadastro
Then o sistema deve recusar a transação sem criar a conta
And deve informar que a data de nascimento deve ser válida e não futura
```

## RF0002: Ativar conta de acesso via token de ativação

**História de usuário:** Como Usuário, Quero ativar minha conta de acesso utilizando o token recebido por email, Para que eu possa confirmar minha identidade e liberar meu acesso completo ao sistema com o status de conta "Ativada".

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Ativar conta de acesso via token de ativação

Scenario: Caminho Feliz - Sucesso na ativação da conta
Given que eu possuo uma conta no banco de dados com o status "Pendente"
And eu possuo um token de ativação válido e gerado há menos de 24 horas
When eu acesso o link ou submeto o token de ativação para o sistema
Then o sistema deve processar o token com sucesso
And deve alterar o status da minha conta de acesso de "Pendente" para "Ativada"

Scenario: Caminho Alternativo 1 - Falha na ativação por token expirado (RN0007)
Given que eu possuo uma conta no banco de dados com o status "Pendente"
And eu possuo um token de ativação gerado há mais de 24 horas
When eu tento submeter este token de ativação para o sistema
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Este link de ativação expirou. Solicite um novo e-mail de ativação."

Scenario: Caminho Alternativo 2 - Falha na ativação por token já utilizado ou inválido (RN0008)
Given que o meu token de ativação já foi utilizado com sucesso anteriormente para ativar a conta
When eu tento submeter este token de ativação invalidado novamente para o sistema
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Link de ativação inválido!"
```

## RF0003: Reenviar e-mail com link de ativação da conta de acesso

**História de usuário:** Como Usuário, Quero solicitar o reenvio do link de ativação para o meu e-mail cadastrado, Para que eu possa obter um novo token válido e concluir a ativação da minha conta com status pendente.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Reenviar e-mail com link de ativação da conta de acesso

Scenario: Caminho Feliz - Sucesso no reenvio do link de ativação
Given que eu sou um usuário com uma conta no banco de dados com o status "Pendente"
And eu informo o meu endereço de e-mail corretamente
When eu solicito o reenvio do link de ativação
Then o sistema deve gerar um novo token de ativação
And deve disparar automaticamente uma nova mensagem para o e-mail cadastrado contendo o link atualizado

Scenario: Caminho Alternativo 1 - Falha no reenvio para e-mail inexistente (RN0009)
Given que eu informo um endereço de e-mail que não consta na base de dados do sistema
When eu solicito o reenvio do link de ativação
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Endereço de e-mail não cadastrado"

Scenario: Caminho Alternativo 2 - Falha no reenvio para conta já ativada (RN0010)
Given que eu informo o endereço de e-mail de uma conta que já possui o status "Ativada"
When eu solicito o reenvio do link de ativação
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Este endereço de e-mail pertence a uma conta ativada."

Scenario: Caminho Alternativo 3 - Falha na ativação por uso de token sobreposto/invalidado (RN0011)
Given que eu solicitei o reenvio do link de ativação e o sistema gerou um novo token para minha conta
When eu tento realizar a ativação utilizando o token antigo que havia sido enviado na primeira mensagem
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Link de ativação inválido!"
```

## RF0004: Autenticar credenciais da conta de acesso e retornar sessão de acesso

**História de usuário:** Como Usuário, Quero autenticar minhas credenciais (e-mail e senha) no sistema, Para que eu possa iniciar uma sessão de acesso segura e utilizar as funcionalidades restritas da minha conta.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Autenticar credenciais da conta de acesso e retornar sessão de acesso

Scenario: Caminho Feliz - Sucesso na autenticação e criação de sessão
Given que eu possuo uma conta cadastrada e com status "Ativada"
And eu informo o meu e-mail e senha corretos na tela de login
When eu submeto o formulário de autenticação
Then o sistema deve validar as credenciais com sucesso
And deve criar uma sessão stateful persistida no servidor e enviar ao cliente um identificador opaco por cookie HttpOnly

Scenario: Caminho Alternativo 1 - Falha de login por credenciais inválidas (RN0012)
Given que eu estou na tela de login
When eu submeto uma combinação de e-mail e/ou senha que não coincide com os dados registrados no banco de dados
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Credenciais inválidas!"

Scenario: Caminho Alternativo 2 - Falha de login por status de acesso pendente (RN0013)
Given que a minha conta cadastrada possui o status atual como "Pendente"
When eu submeto o meu e-mail e senha corretos na tela de login
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Conta de acesso pendente. Ative a conta com o e-mail de verificação enviado anteriormente."

Scenario: Caminho Alternativo 3 - Bloqueio de acesso por sessão ausente, encerrada ou inválida (RN0014)
Given que a minha sessão anterior foi encerrada (por logout explícito ou redefinição de senha)
When eu tento acessar uma rota ou recurso do sistema que exige autenticação
Then o sistema deve identificar a ausência de uma sessão ativa
And deve recusar a transação retornando a mensagem de erro: "Sua sessão é inválida ou foi encerrada. Por favor, faça login novamente."
```

## RF0005: Encerrar sessão de acesso ativa

**História de usuário:** Como Usuário, Quero encerrar a minha sessão de acesso ativa (fazer logout), Para que eu possa garantir a segurança da minha conta e impedir acessos não autorizados no meu dispositivo.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Encerrar sessão de acesso ativa

Scenario: Caminho Feliz - Sucesso no encerramento da sessão ativa
Given que eu sou um usuário com uma sessão de acesso ativa e válida
When eu solicito o encerramento da sessão (logout)
Then o sistema deve identificar a sessão stateful apresentada no cookie HttpOnly da requisição
And deve revogar a sessão correspondente na persistência do servidor e remover o cookie do cliente
And deve garantir que o meu acesso a áreas restritas seja bloqueado até um novo login

Scenario: Caminho Alternativo 1 - Tentativa de encerramento sem sessão ativa
Given que o cookie de sessão está ausente, inválido ou corresponde a uma sessão já revogada
When eu solicito o encerramento da sessão (logout)
Then o sistema deve identificar que não existe uma sessão ativa válida para encerrar
And deve retornar HTTP 401 com uma resposta JSON padronizada de sessão inválida ou encerrada
```

## RF0006: Solicitar redefinição de senha de acesso via e-mail

**História de usuário:** Como Usuário, Quero solicitar a redefinição da minha senha informando meu e-mail cadastrado, Para que eu possa receber um link de verificação seguro e recuperar o acesso à minha conta.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Solicitar redefinição de senha de acesso via e-mail

Scenario: Caminho Feliz - Sucesso na solicitação e disparo de e-mail de redefinição
Given que eu possuo uma conta no banco de dados com o status "Ativada"
And eu informo corretamente o meu endereço de e-mail na tela de recuperação
When eu submeto a solicitação de redefinição de senha
Then o sistema deve gerar um token de verificação com validade estrita de 1 hora
And deve disparar automaticamente uma mensagem para o e-mail cadastrado contendo o link associado ao token

Scenario: Caminho Alternativo 1 - Falha na solicitação por e-mail inexistente (RN0015)
Given que eu estou na tela de recuperação de senha
When eu submeto uma solicitação informando um endereço de e-mail que não está cadastrado no banco de dados
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "E-mail não cadastrado!"

Scenario: Caminho Alternativo 2 - Falha na solicitação por conta pendente (RN0016)
Given que a minha conta cadastrada possui o status atual como "Pendente"
When eu submeto a solicitação de redefinição de senha com o meu e-mail
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Ative a conta com o e-mail de verificação enviado anteriormente para alterar a senha."

Scenario: Caminho Alternativo 3 - Falha no uso de token de redefinição expirado (RN0017)
Given que eu solicitei a redefinição de senha e recebi o e-mail com o link de verificação
And já se passaram mais de 1 hora desde a geração do token
When eu tento acessar o link ou submeter o token expirado para o sistema
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Este link de redefinição expirou. Solicite a redefinição novamente."
```

## RF0007: Redefinir senha de acesso via token de redefinição

**História de usuário:** Como Usuário, Quero redefinir minha senha de acesso utilizando um token de recuperação seguro, Para que eu possa restabelecer meu login e acessar minha conta após ter esquecido minhas credenciais.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Redefinir senha de acesso via token de redefinição

Scenario: Caminho Feliz - Sucesso na redefinição da senha
Given que eu possuo um token de redefinição válido, gerado há menos de 1 hora
And eu acesso a rota de redefinição de senha
When eu preencho os campos de Nova Senha e Confirmação de Nova Senha com "SenhaForte123!"
And submeto a transação contendo o token de redefinição
Then o sistema deve alterar a senha de acesso da minha conta no banco de dados com sucesso
And deve invalidar imediatamente o token utilizado
And deve encerrar todas as minhas sessões de acesso ativas (logout compulsório)

Scenario: Caminho Alternativo 1 - Falha na redefinição por senha fora do padrão de força estrutural (RN0002)
Given que eu possuo um token de redefinição válido
When eu preencho os campos de Nova Senha e Confirmação de Nova Senha com "fraca12"
And submeto a transação contendo o token de redefinição
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro específica para o campo: "Utilize no mínimo 8 caracteres, incluindo pelo menos uma letra maiúscula, uma minúscula, um número e um caractere especial."

Scenario: Caminho Alternativo 2 - Falha na redefinição por uso de token expirado (RN0017)
Given que eu possuo um token de redefinição gerado há mais de 1 hora
When eu preencho as senhas corretamente
And submeto a transação contendo o token de redefinição expirado
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Este link de redefinição expirou. Solicite a redefinição novamente."

Scenario: Caminho Alternativo 3 - Falha na redefinição por token de unicidade já utilizado (RN0018)
Given que o meu token de redefinição já foi utilizado com sucesso anteriormente para alterar a senha
When eu tento submeter uma nova requisição utilizando o mesmo token invalidado
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Link de redefinição inválido!"

Scenario: Caminho Alternativo 4 - Falha na redefinição por token sobreposto (RN0019)
Given que eu solicitei a redefinição de senha duas vezes, gerando dois tokens distintos
When eu tento redefinir a senha utilizando o primeiro token (que foi sobreposto e invalidado pelo segundo)
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Link de redefinição inválido!"

Scenario: Caminho Alternativo 5 - Falha na redefinição por divergência na confirmação de senha (RN0020)
Given que eu possuo um token de redefinição válido
When eu preencho o campo de Nova Senha com "SenhaForte123!"
And preencho o campo de Confirmação de Nova Senha com "SenhaDiferente123!"
And submeto a transação
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Divergência nos valores da senha e confirmação de senha!"
```

## RF0008: Redefinir senha de acesso em sessão ativa

**História de usuário:** Como Usuário, Quero redefinir a minha senha de acesso a partir das configurações da minha conta logada, Para que eu possa atualizar minhas credenciais proativamente e manter a segurança do meu perfil.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Redefinir senha de acesso em sessão ativa

Scenario: Caminho Feliz - Sucesso na redefinição da senha logado
Given que eu sou um usuário com uma sessão de acesso ativa e válida
And eu acesso a área de configurações de segurança da conta
When eu preencho o campo de Senha Atual corretamente
And preencho os campos de Nova Senha e Confirmação de Nova Senha com "NovaSenhaForte123!"
And submeto o formulário de alteração de senha
Then o sistema deve alterar a senha de acesso no banco de dados com sucesso
And deve encerrar a minha sessão de acesso ativa (logout compulsório) exigindo um novo login

Scenario: Caminho Alternativo 1 - Falha na redefinição por divergência da senha atual (RN0021)
Given que eu sou um usuário com uma sessão de acesso ativa
When eu preencho o campo de Senha Atual com uma senha incorreta que não corresponde ao banco de dados
And preencho os campos de Nova Senha e Confirmação de Nova Senha validamente
And submeto o formulário
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Senha atual incorreta!"

Scenario: Caminho Alternativo 2 - Falha na redefinição por nova senha fora do padrão de força estrutural (RN0002)
Given que eu sou um usuário com uma sessão de acesso ativa
When eu preencho o campo de Senha Atual corretamente
And preencho os campos de Nova Senha e Confirmação de Nova Senha com "fraca12"
And submeto o formulário
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro específica para o campo: "Utilize no mínimo 8 caracteres, incluindo pelo menos uma letra maiúscula, uma minúscula, um número e um caractere especial."

Scenario: Caminho Alternativo 3 - Falha na redefinição por divergência na confirmação da nova senha (RN0020)
Given que eu sou um usuário com uma sessão de acesso ativa
When eu preencho o campo de Senha Atual corretamente
And preencho o campo de Nova Senha com "NovaSenhaForte123!"
And preencho o campo de Confirmação de Nova Senha com "DiferenteSenha123!"
And submeto o formulário
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Divergência nos valores da senha e confirmação de senha!"
```

## RF0009: Consultar dados cadastrais da própria conta

**História de usuário:** Como Usuário, Quero consultar os dados cadastrais da minha conta a partir da minha sessão ativa, Para que eu possa visualizar minhas informações.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar dados cadastrais da própria conta

Scenario: Caminho Feliz - Sucesso na consulta dos próprios dados cadastrais
Given que eu sou um usuário com uma sessão de acesso ativa e válida
When eu solicito a consulta dos dados do meu perfil
Then o sistema deve identificar a minha conta exclusivamente pela sessão stateful validada
And deve buscar as informações correspondentes no banco de dados
And deve retornar com sucesso um pacote contendo estritamente o meu nome de usuário, e-mail, nível de acesso e preferência de exibição de conteúdo adulto

Scenario: Caminho Alternativo 1 - Falha de privacidade ao tentar consultar dados de terceiros sem privilégio (RN0022)
Given que eu possuo uma sessão de acesso ativa com o nível de "Usuário Padrão"
When eu tento realizar uma requisição de consulta apontando explicitamente para o identificador de outra conta
Then o sistema deve detectar a tentativa de acesso externo
And deve recusar a transação
And deve retornar a mensagem de erro: "Acesso negado: Você não tem permissão para acessar ou modificar os dados deste perfil."
```

## RF0010: Alterar nome de usuário em sessão ativa

**História de usuário:** Como Usuário, Quero alterar o meu nome de usuário a partir das configurações da minha sessão ativa, Para que eu possa atualizar a minha identificação pública no sistema de forma segura e personalizada.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Alterar nome de usuário em sessão ativa

Scenario: Caminho Feliz - Sucesso na alteração do nome de usuário
Given que eu sou um usuário com uma sessão de acesso ativa e válida
When eu preencho o campo de Novo Nome de Usuário com um valor válido (ex: "novo_user_123")
And submeto a solicitação de alteração
Then o sistema deve identificar a minha conta exclusivamente pela sessão stateful validada
And deve atualizar o meu nome de usuário no banco de dados com sucesso

Scenario: Caminho Alternativo 1 - Falha na alteração por formatação inválida (RN0001)
Given que eu sou um usuário com uma sessão de acesso ativa
When eu preencho o campo de Novo Nome de Usuário com um valor que viola as regras de formatação (ex: "User Inválido!" ou "ab")
And submeto a solicitação de alteração
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro específica para o campo: "Utilize entre 3 e 20 caracteres, sem espaços, acentos ou caracteres especiais."

Scenario: Caminho Alternativo 2 - Falha de privacidade ao tentar alterar o nome de terceiros sem privilégio (RN0022)
Given que eu possuo uma sessão de acesso ativa com o nível de "Usuário Padrão"
When eu tento forçar a requisição de alteração de nome de usuário apontando para o identificador de outra conta
Then o sistema deve detectar a tentativa de modificação externa
And deve recusar a transação imediatamente
And deve retornar a mensagem de erro: "Acesso negado: Você não tem permissão para acessar ou modificar os dados deste perfil."
```

## RF0011: Alterar preferência de exibição de conteúdo adulto

**História de usuário:** Como Usuário, Quero alterar a minha preferência de exibição de conteúdo adulto (ativar ou desativar) nas configurações da minha conta, Para que eu tenha controle absoluto sobre a exibição ou restrição de obras sensíveis (+18) durante a minha navegação no catálogo.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Alterar preferência de exibição de conteúdo adulto

Scenario: Caminho Feliz - Sucesso na alteração da preferência de conteúdo
Given que eu sou um usuário com uma sessão de acesso ativa e válida e possuo data de nascimento válida que indica 18 anos completos
When eu submeto a solicitação de alteração da preferência de conteúdo adulto para "Ativado"
Then o sistema deve identificar a minha conta exclusivamente pela sessão stateful validada
And deve atualizar o estado da minha preferência no banco de dados com sucesso

Scenario: Caminho Alternativo 1 - Falha de privacidade ao tentar alterar preferência de terceiros sem privilégio (RN0022)
Given que eu possuo uma sessão de acesso ativa com o nível de "Usuário Padrão"
When eu tento forçar a requisição de alteração de preferência de exibição apontando para o identificador de outra conta
Then o sistema deve detectar a tentativa de modificação externa não autorizada
And deve recusar a transação imediatamente
And deve retornar a mensagem de erro: "Acesso negado: Você não tem permissão para acessar ou modificar os dados deste perfil."

Scenario: Caminho Alternativo 2 - Bloqueio da ativação para usuário menor de 18 anos (RN0048)
Given que eu possuo uma sessão de acesso ativa e válida
And minha data de nascimento válida indica menos de 18 anos completos
When eu solicito a alteração da preferência de conteúdo adulto para "Ativado"
Then o sistema deve recusar a ativação e manter a preferência desativada
And deve omitir conteúdo adulto das respostas públicas destinadas à minha conta
```

## RF0012: Excluir permanentemente conta de acesso e dados vinculados

**História de usuário:** Como Usuário, Quero excluir permanentemente a minha conta de acesso e todos os meus dados vinculados mediante a confirmação da minha senha, Para que eu possa exercer o meu direito de privacidade e remover minha presença do sistema de forma irreversível.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Excluir permanentemente conta de acesso e dados vinculados

Scenario: Caminho Feliz - Sucesso na exclusão da própria conta
Given que eu sou um usuário com uma sessão de acesso ativa e válida
When eu solicito a exclusão permanente do meu perfil sem informar identificador de usuário na requisição
And preencho o parâmetro obrigatório de "Senha Atual" de forma correta
Then o sistema deve validar a senha atual e identificar a conta exclusivamente pela sessão stateful ativa
And deve excluir permanentemente a minha conta e os dados que dependam exclusivamente dela no banco de dados
And deve encerrar a minha sessão ativa instantaneamente

Scenario: Caminho Alternativo 1 - Falha na exclusão por senha atual incorreta (RN0021)
Given que eu possuo uma sessão de acesso ativa e válida
When eu solicito a exclusão permanente do meu perfil
And preencho o parâmetro de "Senha Atual" com um valor divergente da minha credencial vigente
Then o sistema deve abortar a exclusão para proteger a conta
And deve recusar a transação retornando a mensagem de erro: "Senha atual incorreta!"

Scenario: Caminho Alternativo 2 - Falha de privacidade ao tentar excluir conta de terceiros sem privilégio (RN0022)
Given que eu possuo uma sessão de acesso ativa com o nível de "Usuário Padrão"
When eu tento forçar a requisição de exclusão permanente apontando para o identificador de outra conta
Then o sistema deve detectar a tentativa de exclusão externa não autorizada
And deve recusar a transação imediatamente
And deve retornar a mensagem de erro: "Acesso negado: Você não tem permissão para acessar ou modificar os dados deste perfil."
```

## RF0013: Cadastrar nova Obra no banco de dados

**História de usuário:** Como Administrador, Quero cadastrar uma nova Obra com seus dados estruturais, autorais e editoriais, Para que eu possa expandir o catálogo e prepará-la para posterior publicação na área pública.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Cadastrar nova Obra no banco de dados

Scenario: Caminho Feliz - Sucesso no cadastro de uma nova Obra com todos os dados
Given que eu sou um Administrador autenticado no sistema
And eu preencho todos os dados obrigatórios e opcionais do formulário de nova Obra corretamente
And não existe outra Obra cadastrada com o mesmo título em português
When eu submeto o formulário de cadastro
Then o sistema deve persistir a nova Obra, os múltiplos papéis de autoria e os vínculos editoriais ordenados em uma única transação
And deve atribuir compulsoriamente a visibilidade da Obra como "Privado"

Scenario: Caminho Alternativo 1 - Falha no cadastro por submissão de autor duplicado (RN0023)
Given que eu estou preenchendo o formulário de cadastro de uma nova Obra
When eu vinculo o mesmo "Autor" duas ou mais vezes na mesma requisição
And submeto o formulário
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Autor duplicado!"

Scenario: Caminho Alternativo 2 - Falha no cadastro por título duplicado (RN0025)
Given que já existe uma Obra registrada com o título em português "Fire Force"
When eu tento cadastrar outra Obra com o mesmo título, desconsiderando diferenças entre maiúsculas, minúsculas e espaços externos
Then o sistema deve identificar a duplicidade
And deve recusar a transação retornando o erro: "Uma Obra com esse mesmo título já foi cadastrada anteriormente!"

Scenario: Caminho Alternativo 3 - Falha no cadastro por omissão de dados obrigatórios (RN0026)
Given que eu estou preenchendo o formulário de cadastro de uma nova Obra
When eu omito um ou mais dados obrigatórios, como o "País de Origem" ou a importação de uma capa interna válida
And submeto o formulário
Then o sistema deve recusar a transação preemptivamente
And deve retornar a mensagem de erro: "Dados obrigatórios faltando!"
```

## RF0014: Alterar dados de uma Obra específica no banco de dados

**História de usuário:** Como Administrador, Quero alterar os dados cadastrais de uma Obra específica existente no sistema, Para que eu possa atualizar seu status de publicação, corrigir informações ou enriquecer seus metadados sem perder o histórico do catálogo.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Alterar dados de uma Obra específica no banco de dados

Scenario: Caminho Feliz - Sucesso na alteração parcial de dados da Obra
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And eu acesso a interface de edição de uma Obra específica fornecendo seu identificador único
When eu altero apenas alguns campos permitidos (ex: atualizo o "Status de Publicação Original" para "Completo" e ajusto o "Número de Volumes")
And submeto o formulário de alteração
Then o sistema deve validar a transação com sucesso
And deve atualizar exclusivamente os campos modificados no banco de dados
And deve preservar integralmente os valores anteriores dos campos que não sofreram alteração

Scenario: Caminho Alternativo 1 - Falha na alteração por tentativa de vinculação de autor duplicado (RN0023)
Given que eu estou na interface de alteração de dados de uma Obra
When eu edito a lista de Autores e vinculo um mesmo Autor repetidas vezes na mesma requisição
And submeto o formulário
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Autor duplicado!"
And deve preservar a capa interna já associada quando não houver substituta válida

Scenario: Caminho Alternativo 2 - Falha na alteração por título duplicado (RN0025)
Given que existem duas Obras distintas cadastradas no banco de dados (Obra A e Obra B)
When eu altero o título em português da Obra A para o mesmo título vigente da Obra B
And submeto o formulário
Then o sistema deve detectar a duplicidade de título no catálogo
And deve recusar a transação
And deve retornar a mensagem de erro: "Uma Obra com esse mesmo título já foi cadastrada anteriormente!"

Scenario: Caminho Alternativo 3 - Falha na alteração por remoção de dados obrigatórios (RN0026)
Given que eu estou na interface de alteração de dados de uma Obra
When eu apago ou solicito remover sem substituta válida o conteúdo de um campo estritamente obrigatório, como Título em Pt-br ou a capa interna
And submeto o formulário
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Dados obrigatórios faltando!"
```

## RF0015: Excluir uma Obra específica do banco de dados

**História de usuário:** Como Administrador, Quero excluir permanentemente uma Obra específica do banco de dados através do seu identificador único, Para que eu possa remover registros incorretos ou indesejados, mantendo a integridade e a limpeza do catálogo do sistema.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Excluir uma Obra específica do banco de dados

Scenario: Caminho Feliz - Sucesso na exclusão de uma Obra sem dependências
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Obra alvo da exclusão possui o status de visibilidade como "Privado"
And a Obra não possui nenhuma Edição vinculada a ela no banco de dados
When eu solicito a exclusão permanente da Obra enviando o seu identificador único
Then o sistema deve processar a requisição com sucesso
And deve excluir a Obra permanentemente do banco de dados

Scenario: Caminho Alternativo 1 - Falha na exclusão por proteção de visibilidade pública (RN0028)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Obra alvo da exclusão possui o status de visibilidade como "Público"
When eu solicito a exclusão permanente desta Obra
Then o sistema deve acionar a trava de segurança
And deve recusar a transação
And deve retornar a mensagem de erro: "Essa Obra está pública, não pode ser excluída!"

Scenario: Caminho Alternativo 2 - Falha na exclusão por Edições vinculadas (RN0029)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Obra alvo da exclusão possui o status de visibilidade como "Privado"
And a Obra possui uma ou mais Edições vinculadas a ela no banco de dados
When eu solicito a exclusão permanente desta Obra
Then o sistema deve acionar a restrição de integridade da relação entre Obra e Edição
And deve recusar a exclusão enquanto existirem Edições vinculadas
And deve preservar integralmente a Obra e suas Edições no banco de dados
```

## RF0016: Consultar coleção de Obras cadastradas no banco de dados

**História de usuário:** Como Administrador, Quero consultar a coleção de Obras cadastradas no sistema utilizando filtros combinados e ordenação, Para que eu possa visualizar rapidamente um resumo do acervo, auditar informações e localizar registros específicos para gerenciamento.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar coleção de Obras cadastradas no banco de dados

Scenario: Caminho Feliz - Sucesso na consulta geral com ordenação padrão
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And existem Obras cadastradas no banco de dados
When eu solicito a consulta da coleção de Obras sem aplicar filtros
Then o sistema deve retornar a coleção de registros com sucesso
And a coleção deve estar ordenada de forma crescente pelo Título em Pt-br (A-Z)
And cada registro deve conter URL da Capa, Título e Título Original, Autores ordenados pela prioridade dos papéis, País de Origem, Tipo de Obra, Quantidade calculada de Edições e Status de Visibilidade

Scenario: Caminho Alternativo 1 - Sucesso na consulta aplicando múltiplos filtros combinados
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu solicito a consulta da coleção de Obras
And aplico um termo textual para busca (ex: "Fire")
And aplico os filtros de Tipo de Obra, País de Origem e Status de Visibilidade
Then o sistema deve processar a requisição aplicando a correspondência parcial em Títulos ou Nomes de Autores
And deve retornar estritamente a subcoleção de Obras que satisfaçam todos os critérios combinados simultaneamente

Scenario: Caminho Alternativo 2 - Sucesso na consulta com ordenação invertida
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu solicito a consulta da coleção de Obras
And exijo explicitamente a ordenação invertida
Then o sistema deve retornar a coleção de registros ordenada de forma decrescente pelo Título em Pt-br (Z-A)

Scenario: Caminho Alternativo 3 - Consulta com filtros restritivos resultando em coleção vazia (Empty State)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu aplico uma combinação de filtros ou um termo textual que não corresponde a nenhuma Obra cadastrada no banco de dados
Then o sistema deve processar a requisição com sucesso
And deve retornar uma coleção vazia (array vazio) sem disparar exceções de erro de servidor
```

## RF0017: Consultar dados de uma Obra específica do banco de dados

**História de usuário:** Como Administrador, Quero consultar detalhadamente todos os dados de uma Obra específica através do seu identificador único, Para que eu possa visualizar integralmente o registro, auditar suas informações e garantir que visualizo o estado mais recente salvo no banco de dados.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar dados de uma Obra específica do banco de dados

Scenario: Caminho Feliz - Sucesso na consulta de um registro existente
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And existe uma Obra cadastrada no banco de dados com um identificador único válido
When eu solicito a consulta detalhada enviando este identificador único
Then o sistema deve processar a requisição com sucesso
And deve retornar o payload completo contendo todos os campos preenchidos da Obra os dados retornados devem refletir estritamente o estado mais recente salvo no banco de dados

Scenario: Caminho Alternativo 1 - Falha na consulta por identificador inexistente (Not Found)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu solicito a consulta detalhada enviando um identificador único que não consta no banco de dados
Then o sistema deve interceptar a ausência do registro
And deve recusar a transação
And deve retornar um código de erro apropriado (ex: HTTP 404) com a mensagem: "Obra não encontrada!"
```

## RF0018: Alterar status de visibilidade de uma Obra específica do banco de dados

**História de usuário:** Como Administrador, Quero alterar o status de visibilidade de uma Obra específica (entre "Público" e "Privado"), Para que eu possa controlar a disponibilidade do registro no catálogo aberto.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Alterar status de visibilidade de uma Obra específica do banco de dados

Scenario: Caminho Feliz - Sucesso na alteração do status de Privado para Público
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Obra alvo da alteração possui o status de visibilidade atual como "Privado"
When eu solicito a alteração do status de visibilidade para "Público" enviando o identificador único da Obra
Then o sistema deve processar a requisição com sucesso
And deve atualizar o status de visibilidade da Obra para "Público" no banco de dados

Scenario: Caminho Feliz - Sucesso na alteração do status de Público para Privado (Sem dependências conflitantes)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Obra alvo possui o status de visibilidade atual como "Público"
And todas as Edições vinculadas a esta Obra já possuem o status de visibilidade "Privado" (ou a Obra não possui Edições cadastradas)
When eu solicito a alteração do status de visibilidade da Obra para "Privado"
Then o sistema deve processar a requisição com sucesso
And deve atualizar o status de visibilidade da Obra para "Privado" no banco de dados

Scenario: Caminho Alternativo 1 - Falha na alteração para Privado devido a Edições Públicas ativas (RN0030)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Obra alvo possui o status de visibilidade atual como "Público" a Obra possui uma ou mais Edições vinculadas a ela que ostentam o status de visibilidade "Público"
When eu solicito a alteração do status de visibilidade da Obra para "Privado"
Then o sistema deve acionar a trava de integridade hierárquica
And deve recusar a transação
And deve retornar a mensagem de erro: "Essa Obra possui Edições públicas, não pode ser rebaixada para privada!"
```

## RF0019: Cadastrar nova Edição vinculada a uma Obra no banco de dados

**História de usuário:** Como Administrador, Quero cadastrar uma nova Edição física vinculada a uma Obra matriz existente, Para que eu possa catalogar os diferentes formatos de publicação editorial (ex: Tankobon, Kanzenban) lançados no mercado brasileiro.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Cadastrar nova Edição vinculada a uma Obra no banco de dados

Scenario: Caminho Feliz - Sucesso no cadastro da edição e atribuição de status (RN0034)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And eu acesso a interface de cadastro de Edição vinculando-a ao identificador único de uma Obra matriz
When eu preencho Editora Brasileira, Tipo de Edição, Acabamento, Formato, Número cronológico, Status de publicação no Brasil e informo uma URL HTTPS válida para importação da capa interna
And submeto o formulário de cadastro
Then o sistema deve gravar os dados da nova Edição no banco de dados com sucesso, vinculando-a à Obra matriz
And deve atribuir compulsoriamente a visibilidade desta nova Edição como "Privado"

Scenario: Caminho Alternativo 1 - Falha no cadastro por colisão de chave cronológica (RN0032)
Given que a Obra matriz já possui uma Edição cadastrada no banco de dados sob o "Número Cronológico da Edição" equivalente a "1ª Edição"
When eu tento cadastrar uma nova Edição vinculada a esta exata mesma Obra matriz informando o "Número Cronológico da Edição" também como "1ª Edição"
And submeto o formulário
Then o sistema deve detectar a quebra de unicidade da chave composta (Obra ID + Número Cronológico)
And deve recusar a transação
And deve retornar a mensagem de erro: "Uma Edição com esse mesmo número cronológico já foi cadastrada anteriormente!"

Scenario: Caminho Alternativo 2 - Falha no cadastro por ausência de dados obrigatórios (RN0033)
Given que eu estou na interface de cadastro de nova Edição vinculada a uma Obra matriz
When eu submeto o formulário omitindo um campo obrigatório, como Formato, Editora Brasileira ou a importação de uma capa interna válida
Then o sistema deve recusar a transação deve retornar a mensagem de erro: "Dados obrigatórios faltando!"
```

## RF0020: Alterar dados de uma Edição específica no banco de dados

**História de usuário:** Como Administrador, Quero alterar os dados cadastrais de uma Edição específica existente no sistema, Para que eu possa atualizar seu status de publicação, corrigir informações de formato ou enriquecer seus metadados sem perder o histórico do catálogo da Obra matriz.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Alterar dados de uma Edição específica no banco de dados

Scenario: Caminho Feliz - Sucesso na alteração parcial de dados da Edição
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And eu acesso a interface de edição de uma Edição específica fornecendo seu identificador único
When eu altero apenas alguns campos permitidos (ex: atualizo o "Status de Publicação no Brasil" para "Concluído")
And submeto o formulário de alteração
Then o sistema deve validar a transação com sucesso
And deve atualizar exclusivamente os campos modificados no banco de dados
And deve preservar integralmente os valores anteriores dos campos que não sofreram alteração

Scenario: Caminho Alternativo 1 - Falha na alteração por colisão de chave cronológica na mesma Obra (RN0032)
Given que existem duas Edições distintas cadastradas no banco de dados para a mesma Obra matriz (ex: 1ª Edição e 2ª Edição)
When eu edito os dados da "2ª Edição" e altero o seu campo "Número Cronológico da Edição" para "1ª Edição"
And submeto o formulário
Then o sistema deve detectar a quebra de unicidade da chave composta (Identificador Único da Obra matriz + Número Cronológico da Edição)
And deve recusar a transação
And deve retornar a mensagem de erro: "Uma Edição com esse mesmo número cronológico já foi cadastrada anteriormente!"
And deve preservar a capa interna já associada quando não houver substituta válida

Scenario: Caminho Alternativo 2 - Falha na alteração por remoção de dados obrigatórios (RN0033)
Given que eu estou na interface de alteração de dados de uma Edição específica
When eu removo o conteúdo de um campo obrigatório, como Editora Brasileira, Formato ou solicito remover a capa interna sem substituta válida
And submeto o formulário
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Dados obrigatórios faltando!"
```

## RF0021: Excluir uma Edição específica do banco de dados

**História de usuário:** Como Administrador, Quero excluir permanentemente uma Edição específica do banco de dados através do seu identificador único, Para que eu possa remover cadastros errôneos ou descontinuados, mantendo a integridade estrutural e a higienização do catálogo de publicações.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Excluir uma Edição específica do banco de dados

Scenario: Caminho Feliz - Sucesso na exclusão de uma Edição sem dependências
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Edição alvo da exclusão possui o status de visibilidade como "Privado"
And a Edição não possui nenhum Volume vinculado a ela no banco de dados
When eu solicito a exclusão permanente da Edição enviando o seu identificador único
Then o sistema deve processar a requisição com sucesso
And deve excluir a Edição permanentemente do banco de dados

Scenario: Caminho Alternativo 1 - Falha na exclusão por proteção de visibilidade pública (RN0035)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Edição alvo da exclusão possui o status de visibilidade como "Público"
When eu solicito a exclusão permanente desta Edição
Then o sistema deve acionar a trava de segurança
And deve recusar a transação
And deve retornar a mensagem de erro: "Essa Edição está pública, não pode ser excluída!"

Scenario: Caminho Alternativo 2 - Falha na exclusão por Volumes vinculados (RN0036)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Edição alvo da exclusão possui o status de visibilidade como "Privado"
And a Edição possui um ou mais Volumes vinculados a ela no banco de dados
When eu solicito a exclusão permanente desta Edição enviando o seu identificador único
Then o sistema deve acionar a restrição de integridade da relação entre Edição e Volume
And deve recusar a exclusão enquanto existirem Volumes vinculados
And deve preservar integralmente a Edição e seus Volumes no banco de dados
```

## RF0022: Consultar coleção de Edições vinculadas a uma Obra no banco de dados

**História de usuário:** Como Administrador, Quero consultar a coleção de Edições cadastradas e vinculadas a uma Obra matriz específica, Para que eu possa visualizar rapidamente um resumo dos formatos de publicação lançados, monitorar a quantidade de volumes atrelados e gerenciar o acervo de forma organizada.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar coleção de Edições vinculadas a uma Obra no banco de dados

Scenario: Caminho Feliz - Sucesso na consulta geral de Edições ordenadas
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And existe uma Obra cadastrada com uma ou mais Edições vinculadas a ela no banco de dados
When eu solicito a consulta da coleção de Edições informando o identificador único da Obra matriz
Then o sistema deve processar a requisição com sucesso
And deve retornar a coleção de registros ordenada de forma decrescente pelo "Número Cronológico da Edição"
And cada registro deve conter URL da Capa da Edição, Editora Brasileira, Número cronológico em formato ordinal, Tipo de Edição, Quantidade calculada de Volumes e Status de Visibilidade

Scenario: Caminho Alternativo 1 - Consulta de Obra sem Edições cadastradas (Empty State)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And existe uma Obra cadastrada validamente, porém sem nenhuma Edição vinculada a ela no momento
When eu solicito a consulta da coleção de Edições informando o identificador desta Obra
Then o sistema deve processar a requisição com sucesso
And deve retornar uma coleção vazia (array vazio) sem disparar exceções de erro de servidor
```

## RF0023: Consultar dados de uma Edição específica do banco de dados

**História de usuário:** Como Administrador, Quero consultar detalhadamente todos os dados de uma Edição específica através do seu identificador único, Para que eu possa visualizar integralmente o registro físico, auditar suas informações e garantir que o payload reflita o estado mais recente no banco de dados.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar dados de uma Edição específica do banco de dados

Scenario: Caminho Feliz - Sucesso na consulta de um registro existente
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And existe uma Edição cadastrada no banco de dados com um identificador único válido
When eu solicito a consulta detalhada enviando este identificador único
Then o sistema deve processar a requisição com sucesso
And deve retornar o payload completo contendo todos os metadados preenchidos da Edição
And os dados retornados devem refletir estritamente o estado mais recente salvo no banco de dados

Scenario: Caminho Alternativo 1 - Falha na consulta por identificador inexistente (Not Found)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu solicito a consulta detalhada enviando um identificador único de Edição que não consta no banco de dados
Then o sistema deve interceptar a ausência do registro
And deve recusar a transação
And deve retornar um código de erro apropriado (ex: HTTP 404) com a mensagem: "Edição não encontrada!"

Scenario: Caminho Alternativo 2 - Falha na consulta por formato de identificador inválido (Bad Request)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu solicito a consulta detalhada enviando um identificador que não seja um número inteiro válido
Then o sistema deve recusar a transação antes de efetuar a busca
And deve retornar a mensagem de erro: "Formato de identificador inválido."
```

## RF0024: Alterar status de visibilidade de uma Edição específica do banco de dados

**História de usuário:** Como Administrador, Quero alterar o status de visibilidade de uma Edição específica (entre "Público" e "Privado"), Para que eu possa controlar a sua disponibilidade no catálogo aberto aos usuários e garantir que esteja alinhada às regras de publicação da Obra matriz.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Alterar status de visibilidade de uma Edição específica do banco de dados

Scenario: Caminho Feliz - Sucesso na alteração para "Público" com cascata para os Volumes (RN0040)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Edição alvo possui o status de visibilidade "Privado"
And a Obra matriz vinculada a esta Edição possui o status de visibilidade "Público"
When eu solicito a alteração do status de visibilidade da Edição para "Público"
Then o sistema deve processar a requisição com sucesso atualizando o status da Edição para "Público"
And deve alterar compulsoriamente o status de visibilidade de todos os Volumes vinculados a esta Edição para "Público"

Scenario: Caminho Feliz - Sucesso na alteração para "Privado" com cascata para os Volumes (RN0040)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Edição alvo possui o status de visibilidade "Público"
And nenhum dos Volumes vinculados a esta Edição está presente em Estantes Digitais ou Listas de Desejos de usuários
When eu solicito a alteração do status de visibilidade da Edição para "Privado"
Then o sistema deve processar a requisição com sucesso atualizando o status da Edição para "Privado"
And deve alterar compulsoriamente o status de visibilidade de todos os Volumes vinculados a esta Edição para "Privado"

Scenario: Caminho Alternativo 1 - Falha ao tentar tornar "Público" uma Edição de Obra "Privada" (RN0037)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Edição alvo possui o status de visibilidade "Privado"
And a Obra matriz vinculada a esta Edição também está com o status "Privado"
When eu solicito a alteração do status de visibilidade da Edição para "Público"
Then o sistema deve acionar a trava de dependência hierárquica superior
And deve recusar a transação
And deve retornar a mensagem de erro: "Essa Edição está vinculada a uma Obra privada, não pode ser publicada!"

Scenario: Caminho Alternativo 2 - Falha ao tentar tornar "Privado" uma Edição com Volumes em uso (RN0042)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And a Edição alvo possui o status de visibilidade "Público"
And pelo menos um Volume vinculado a esta Edição está salvo em Estantes Digitais ou Listas de Desejos de algum usuário
When eu solicito a alteração do status de visibilidade da Edição para "Privado"
Then o sistema deve acionar a trava de proteção de uso
And deve recusar a transação
And deve retornar a mensagem de erro: "Essa Edição tem Volumes vinculados a usuários, não pode ser tornada privada!"
```

## RF0025: Cadastrar novo Volume vinculado a uma Edição no banco de dados

**História de usuário:** Como Administrador, Quero cadastrar um novo Volume físico vinculado a uma Edição existente, Para que eu possa catalogar individualmente cada tomo com capa, data, preço e identificadores próprios.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Cadastrar novo Volume vinculado a uma Edição no banco de dados

Scenario: Caminho Feliz - Sucesso no cadastro do volume e herança compulsória de status (RN0040)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And eu acesso a interface de cadastro de Volume informando o identificador único válido de uma Edição matriz
When eu preencho Número do Volume, URL HTTPS para importação da capa interna, Precisão da data e Data de publicação com valores válidos
And opcionalmente informo Volume Único, Moeda, Preço, Páginas, ISBN-10, ISBN-13, Link afiliado e Sinopse
And submeto o formulário de cadastro
Then o sistema deve gravar os dados do novo Volume no banco de dados com sucesso, vinculando-o à Edição matriz
And deve atribuir compulsoriamente a este novo Volume o exato mesmo status de visibilidade vigente na sua Edição matriz (Público ou Privado)

Scenario: Caminho Alternativo 1 - Falha no cadastro por colisão de número na mesma Edição (RN0038)
Given que a Edição matriz já possui um Volume cadastrado no banco de dados com o "Número Sequencial do volume" igual a "1"
When eu tento cadastrar um novo Volume vinculado a esta exata mesma Edição matriz informando o "Número Sequencial do volume" também como "1"
And submeto o formulário
Then o sistema deve detectar a quebra de unicidade da chave composta (Identificador Único da Edição matriz + Número do Volume)
And deve recusar a transação
And deve retornar a mensagem de erro: "Um volume desta edição com esse mesmo número já foi cadastrado anteriormente!"

Scenario: Caminho Alternativo 2 - Falha no cadastro por ausência de dados obrigatórios (RN0039)
Given que eu estou na interface de cadastro de novo Volume vinculado a uma Edição matriz
When eu submeto o formulário sem Número do Volume, uma capa interna válida ou Data de publicação válida
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Dados obrigatórios faltando!"
```

## RF0026: Alterar dados de um Volume específico no banco de dados

**História de usuário:** Como Administrador, Quero alterar parcial ou totalmente os dados cadastrais de um Volume específico existente no sistema, Para que eu possa atualizar informações de preço, lançamento ou corrigir dados físicos sem perder o histórico geral da publicação.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Alterar dados de um Volume específico no banco de dados

Scenario: Caminho Feliz - Sucesso na alteração parcial de dados do Volume
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And eu acesso a interface de edição de um Volume específico fornecendo seu identificador único
When eu altero apenas alguns campos (ex: atualizo o "Preço de Capa" e adiciono uma "Sinopse do volume")
And submeto o formulário de alteração
Then o sistema deve validar a transação com sucesso
And deve atualizar exclusivamente os campos modificados no banco de dados
And deve preservar integralmente os valores anteriores dos campos que não sofreram alteração

Scenario: Caminho Alternativo 1 - Falha na alteração por colisão de número na mesma Edição (RN0038)
Given que existem dois Volumes distintos cadastrados no banco de dados para a mesma Edição matriz (ex: Volume 1 e Volume 2)
When eu edito os dados do "Volume 2" e altero o seu campo "Número Sequencial do volume" para "1"
And submeto o formulário
Then o sistema deve detectar a quebra de unicidade da chave composta (Identificador Único da Edição matriz + Número do Volume)
And deve recusar a transação
And deve retornar a mensagem de erro: "Um volume desta edição com esse mesmo número já foi cadastrado anteriormente!"
And deve preservar a capa interna já associada quando não houver substituta válida

Scenario: Caminho Alternativo 2 - Falha na alteração por remoção de dados obrigatórios (RN0039)
Given que eu estou na interface de alteração de dados de um Volume específico
When eu removo o Número do Volume, a capa interna sem substituta válida ou a Data de publicação obrigatória
And submeto o formulário
Then o sistema deve recusar a transação
And deve retornar a mensagem de erro: "Dados obrigatórios faltando!"
```

## RF0027: Excluir um Volume específico do banco de dados

**História de usuário:** Como Administrador, Quero excluir permanentemente um Volume específico do banco de dados através do seu identificador único, Para que eu possa remover cadastros incorretos ou duplicados, mantendo a precisão e a integridade do detalhamento físico das edições no catálogo.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Excluir um Volume específico do banco de dados

Scenario: Caminho Feliz - Sucesso na exclusão de um Volume privado
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And o Volume alvo da exclusão possui o status de visibilidade como "Privado"
When eu solicito a exclusão permanente deste Volume enviando o seu identificador único
Then o sistema deve processar a requisição com sucesso
And deve excluir o Volume permanentemente do banco de dados

Scenario: Caminho Alternativo 1 - Falha na exclusão por proteção de visibilidade pública (RN0041)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And o Volume alvo da exclusão possui o status de visibilidade como "Público"
When eu solicito a exclusão permanente deste Volume
Then o sistema deve acionar a trava de segurança do status
And deve recusar a transação
And deve retornar a mensagem de erro exata: "Esse Volume está público, não pode ser excluído!"
```

## RF0028: Consultar coleção de Volumes vinculados a uma Edição no banco de dados

**História de usuário:** Como Administrador, Quero consultar a coleção de Volumes cadastrados e vinculados a uma Edição matriz específica, Para que eu possa visualizar a listagem de tomos físicos pertencentes àquela publicação de forma ordenada e resumida.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar coleção de Volumes vinculados a uma Edição no banco de dados

Scenario: Caminho Feliz - Sucesso na consulta da coleção de Volumes ordenada
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And existe uma Edição cadastrada com um ou mais Volumes vinculados a ela no banco de dados
When eu solicito a consulta da coleção de Volumes informando o identificador único da Edição matriz
Then o sistema deve processar a requisição com sucesso
And deve retornar a coleção de registros ordenada de forma crescente pelo "Número do Volume"
And cada registro deve conter URL da Capa, Número do Volume ou indicação de Volume Único, Data de publicação quando disponível e Status de Visibilidade

Scenario: Caminho Alternativo 1 - Consulta de Edição sem Volumes cadastrados (Empty State)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And existe uma Edição cadastrada validamente, porém sem nenhum Volume vinculado a ela no momento
When eu solicito a consulta da coleção de Volumes informando o identificador desta Edição
Then o sistema deve processar a requisição com sucesso
And deve retornar uma coleção vazia (array vazio) sem disparar exceções de erro de servidor

Scenario: Caminho Alternativo 2 - Falha na consulta por identificador de Edição inexistente (Not Found)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu solicito a consulta da coleção de Volumes informando um identificador de Edição matriz que não existe no banco de dados
Then o sistema deve interceptar a requisição
And deve recusar a transação
And deve retornar um código de erro apropriado (ex: HTTP 404) indicando que a Edição matriz não foi encontrada
```

## RF0029: Consultar dados de um Volume específico do banco de dados

**História de usuário:** Como Administrador, Quero consultar detalhadamente todos os dados de um Volume específico através do seu identificador único, Para que eu possa visualizar integralmente as informações daquela publicação física e garantir que o retorno reflita o estado mais recente salvo no banco de dados.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar dados de um Volume específico do banco de dados

Scenario: Caminho Feliz - Sucesso na consulta detalhada de um registro existente
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And existe um Volume cadastrado no banco de dados com um identificador único válido
When eu solicito a consulta detalhada enviando este identificador único
Then o sistema deve processar a requisição com sucesso
And deve retornar o payload completo contendo todos os metadados preenchidos do Volume
And os dados retornados devem refletir estritamente o estado mais recente salvo no banco de dados

Scenario: Caminho Alternativo 1 - Falha na consulta por identificador inexistente (Not Found)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu solicito a consulta detalhada enviando um identificador único de Volume que não consta no banco de dados
Then o sistema deve interceptar a ausência do registro
And deve recusar a transação
And deve retornar um código de erro apropriado (ex: HTTP 404) com a mensagem: "Volume não encontrado!"

Scenario: Caminho Alternativo 2 - Falha na consulta por formato de identificador inválido (Bad Request)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu solicito a consulta detalhada enviando um identificador que não seja um número inteiro válido
Then o sistema deve recusar a transação antes de efetuar a busca no banco
And deve retornar a mensagem de erro: "Formato de identificador inválido."
```

## RF0030: Cadastrar novo valor em uma lista de valores précadastrados

**História de usuário:** Como Administrador, Quero cadastrar um ou mais valores em uma categoria administrativa, Para que eu possa manter atualizadas as opções utilizadas nos formulários do catálogo.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Cadastrar novo valor em uma lista de valores pré-cadastrados

Scenario: Caminho Feliz - Sucesso no cadastro de um novo valor referencial
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And acesso a interface de gerenciamento de listas de valores (domínios)
When eu seleciono uma Categoria administrativa editável
And informo um ou mais valores válidos e, quando exigido, os Países de origem relacionados
And submeto o formulário de inclusão
Then o sistema deve registrar todos os novos valores na categoria correspondente em uma única operação
And deve disponibilizá-los nos formulários compatíveis, sem expor como editáveis os campos nativos do sistema

Scenario: Caminho Alternativo 1 - Falha no cadastro por duplicidade de valor na lista (RN0043)
Given que a Categoria alvo selecionada (ex: "Gêneros") já possui o valor "Slice of Life" salvo no banco de dados
When eu tento cadastrar um novo registro nesta mesma Categoria enviando o texto idêntico "Slice of Life"
And submeto a transação
Then o sistema deve detectar a quebra de unicidade dentro da lista específica
And deve recusar a transação
And deve retornar a mensagem de erro exata: "Essa lista já tem esse valor cadastrado!"

Scenario: Caminho Alternativo 2 - Falha no cadastro por ausência de parâmetros obrigatórios
Given que eu estou na interface de gerenciamento de listas de valores
When eu submeto a requisição omitindo a Categoria alvo ou deixando o texto do novo valor em branco
Then o sistema deve recusar a transação antes da inserção no banco de dados
And deve retornar um erro de validação informando que os parâmetros de Categoria e texto do valor são obrigatórios
```

## RF0031: Alterar um valor específico de uma lista de valores précadastrados

**História de usuário:** Como Administrador, Quero alterar o texto de um valor específico pertencente a uma lista de valores pré-cadastrados (informando o seu identificador único), Para que eu possa corrigir erros de digitação ou atualizar nomenclaturas de categorias sem quebrar os vínculos dos registros (Obras/Edições) que já utilizam esse valor.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Alterar um valor específico de uma lista de valores pré-cadastrados

Scenario: Caminho Feliz - Sucesso na alteração do texto do valor
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And eu acesso a interface de edição de um valor pré-cadastrado fornecendo o seu identificador único
When eu preencho o campo de texto com a nova nomenclatura desejada (ex: corrigindo "Sci-fi" para "Ficção Científica")
And submeto o formulário de alteração
Then o sistema deve processar a requisição com sucesso
And deve atualizar o texto e, quando aplicável, os Países relacionados, mantendo o identificador do valor

Scenario: Caminho Alternativo 1 - Falha na alteração por geração de duplicidade na lista (RN0043)
Given que a lista da categoria (ex: "Gêneros") já possui o valor "Ação" previamente cadastrado no banco de dados
When eu edito o texto de um outro registro diferente e tento submetê-lo com a nomenclatura exata de "Ação"
Then o sistema deve detectar a quebra de unicidade dentro da lista de valores
And deve recusar a transação
And deve retornar a mensagem de erro: "Essa lista já tem esse valor cadastrado!"

Scenario: Caminho Alternativo 2 - Falha na alteração por envio de dados incompletos
Given que eu estou na interface de edição de um valor pré-cadastrado
When eu apago o texto e submeto a requisição enviando apenas o identificador único, com o novo valor em branco
Then o sistema deve recusar a transação antes da persistência no banco
And deve retornar um erro de validação informando que o texto do novo valor é obrigatório
```

## RF0032: Excluir um valor específico de uma lista de valores précadastrados

**História de usuário:** Como Administrador, Quero excluir permanentemente um valor específico de uma lista de valores pré-cadastrados, Para que eu possa remover categorias obsoletas ou cadastradas incorretamente, mantendo as opções de domínio limpas e relevantes para a catalogação do acervo.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Excluir um valor específico de uma lista de valores pré-cadastrados

Scenario: Caminho Feliz - Sucesso na exclusão de um valor sem dependências
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And o valor alvo da exclusão (ex: um Gênero ou Editora) não possui nenhum vínculo com registros de Obras, Edições ou Volumes no banco de dados
When eu solicito a exclusão permanente deste valor enviando o seu identificador único
Then o sistema deve processar a requisição com sucesso
And deve excluir o valor permanentemente da respectiva lista no banco de dados

Scenario: Caminho Alternativo 1 - Falha na exclusão por proteção de integridade referencial (RN0044)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And o valor alvo da exclusão está vinculado a pelo menos um registro ativo de Obra, Edição ou Volume
When eu solicito a exclusão permanente deste valor
Then o sistema deve acionar a trava de proteção de uso de domínio
And deve recusar a transação imediatamente
And deve retornar a mensagem de erro exata: "Esse valor está vinculado a um mangá, não pode ser excluído!"
```

## RF0033: Consultar coleção de valores de listas pré-cadastradas

**História de usuário:** Como Administrador, Quero consultar a coleção de valores ativos de uma lista de domínio específica (como Gêneros, Editoras ou Formatos), Para que eu possa visualizar as opções padronizadas disponíveis no sistema e garantir que as interfaces de cadastro de mangás consumam dados consistentes e ordenados.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar coleção de valores de listas pré-cadastradas

Scenario: Caminho Feliz - Sucesso na consulta de valores com ordenação alfabética nativa
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And existe uma Categoria válida (ex: "Tipos de Capa") com valores ativos cadastrados no banco de dados
When eu solicito a consulta da coleção de valores informando a Categoria desejada
Then o sistema deve processar a requisição com sucesso
And deve retornar o identificador, o texto e, quando aplicável, os Países relacionados de cada valor
And deve permitir busca textual, paginação e ordenação alfabética crescente ou decrescente

Scenario: Caminho Alternativo 1 - Consulta de Categoria sem valores cadastrados (Empty State)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And informo uma Categoria válida, porém que não possui nenhum valor cadastrado no momento
When eu solicito a consulta da coleção de valores
Then o sistema deve processar a requisição com sucesso
And deve retornar uma coleção vazia (array vazio) sem disparar exceções de erro de servidor

Scenario: Caminho Alternativo 2 - Falha na consulta por Categoria inexistente ou inválida (Bad Request / Not Found)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu solicito a consulta da coleção de valores enviando um parâmetro de Categoria não mapeado ou inexistente no banco de dados
Then o sistema deve interceptar a requisição
And deve recusar a transação
And deve retornar um código de erro apropriado indicando que a Categoria solicitada não foi encontrada
```

## RF0034: Consultar coleção de contas de usuário cadastradas no banco de dados

**História de usuário:** Como Administrador, Quero consultar a coleção de contas de usuário cadastradas no sistema utilizando filtros combinados e ordenação, Para que eu possa auditar a base de cadastros, monitorar os privilégios concedidos e localizar rapidamente contas específicas para gerenciamento.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar coleção de contas de usuário cadastradas no banco de dados

Scenario: Caminho Feliz - Sucesso na consulta geral com ordenação padrão
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And existem contas de usuário cadastradas no banco de dados
When eu solicito a consulta da coleção de usuários sem aplicar filtros
Then o sistema deve retornar a coleção de registros com sucesso
And a coleção deve estar ordenada de forma crescente pelo Nome de Usuário (A-Z)
And cada registro deve conter Identificador, Nome de Usuário, Endereço de E-mail, Nível de Acesso e Status entre Pendente, Ativada e Bloqueada

Scenario: Caminho Alternativo 1 - Sucesso na consulta aplicando múltiplos filtros combinados
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu solicito a consulta da coleção de usuários
And aplico um termo textual para busca (ex: "dev")
And aplico os filtros de Nível de Acesso (ex: "Usuário Padrão") e Status de Acesso (ex: "Pendente")
Then o sistema deve processar a requisição aplicando a correspondência parcial no Nome de Usuário ou Endereço de E-mail
And deve retornar estritamente a subcoleção de usuários que satisfaçam todos os critérios combinados simultaneamente

Scenario: Caminho Alternativo 2 - Sucesso na consulta com ordenação invertida
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu solicito a consulta da coleção de usuários
And exijo explicitamente a ordenação invertida
Then o sistema deve retornar a coleção de registros ordenada de forma decrescente pelo Nome de Usuário (Z-A)

Scenario: Caminho Alternativo 3 - Consulta com filtros restritivos resultando em coleção vazia (Empty State)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
When eu aplico uma combinação de filtros ou um termo textual que não corresponde a nenhuma conta de usuário cadastrada
Then o sistema deve processar a requisição com sucesso
And deve retornar uma coleção vazia (array vazio) sem disparar exceções de erro de servidor
```

## RF0035: Alterar nível de acesso de uma conta de usuário específica

**História de usuário:** Como Administrador, Quero alterar o nível de acesso de uma conta de usuário específica informando o seu identificador único e o novo nível desejado, Para que eu possa conceder privilégios de gestão a novos membros ou revogar acessos administrativos de terceiros, garantindo o controle seguro da plataforma.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Alterar nível de acesso de uma conta de usuário específica

Scenario: Caminho Feliz - Sucesso na alteração do nível de acesso de uma conta de terceiros
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And o identificador único da conta alvo pertence a um outro usuário (diferente da minha própria conta logada)
When eu solicito a alteração do nível de acesso desta conta informando o novo nível desejado (ex: de "Usuário Padrão" para "Administrador" ou vice-versa)
Then o sistema deve processar a requisição com sucesso
And deve atualizar o nível de acesso da conta alvo no banco de dados para o novo nível informado

Scenario: Caminho Alternativo 1 - Falha na alteração por tentativa de automodificação de privilégios (RN0045)
Given que eu possuo uma sessão ativa com privilégio de "Administrador"
And o identificador único da conta alvo corresponde à minha própria conta administrativa
When eu tento alterar o nível de acesso da minha própria conta
Then o sistema deve acionar a política de prevenção de automodificação
And deve recusar a transação imediatamente
And deve retornar a mensagem de erro exata: "Você não pode alterar o nível de acesso de sua própria conta!"
```

## RF0036: Consultar vitrine pública de Obras

**História de usuário:** Como visitante ou usuário autenticado, Quero consultar a vitrine pública de Obras por busca e filtros, Para que eu possa localizar títulos compatíveis com meus interesses.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar vitrine pública de Obras

Scenario: Caminho Feliz - Consulta paginada da vitrine de Obras
Given que existem Obras públicas cadastradas no catálogo
When eu acesso a vitrine de Obras sem informar filtros
Then o sistema deve retornar uma listagem paginada contendo somente Obras públicas
And cada resultado deve apresentar os metadados públicos de resumo da Obra

Scenario: Caminho Alternativo 1 - Busca e filtros combinados por interseção (RN0047)
Given que eu estou na vitrine pública de Obras
When eu informo um termo de Título ou Autor e seleciono múltiplos filtros de classificação
Then o sistema deve retornar somente as Obras que atendam simultaneamente a todos os critérios

Scenario: Caminho Alternativo 2 - Omissão de conteúdo adulto (RN0048)
Given que eu sou visitante ou possuo a preferência de conteúdo adulto desativada
When eu consulto a vitrine de Obras
Then o sistema deve omitir as Obras classificadas como conteúdo adulto
```

## RF0037: Consultar vitrine pública de Edições

**História de usuário:** Como visitante ou usuário autenticado, Quero consultar a vitrine pública de Edições físicas, Para que eu possa encontrar publicações brasileiras por dados da Edição ou de sua Obra matriz.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar vitrine pública de Edições

Scenario: Caminho Feliz - Busca híbrida de Edições públicas
Given que existem Edições públicas vinculadas a Obras públicas
When eu pesquiso por Título, Título Original ou Autor da Obra matriz
Then o sistema deve retornar a listagem paginada das Edições correspondentes
And não deve retornar Edições privadas nem Edições de Obras privadas

Scenario: Caminho Alternativo 1 - Filtragem por dados da Edição
Given que eu estou na vitrine pública de Edições
When eu seleciono filtros de Editora Brasileira, Formato ou Acabamento
Then o sistema deve aplicar os filtros simultaneamente aos resultados públicos

Scenario: Caminho Alternativo 2 - Preservação do contexto da pesquisa (RN0049)
Given que eu pesquisei por um Título ou Autor na vitrine de Obras
When eu alterno para a vitrine de Edições
Then o sistema deve preservar o termo compatível e reaplicar a pesquisa
```

## RF0038: Consultar detalhes públicos de uma Obra

**História de usuário:** Como visitante ou usuário autenticado, Quero visualizar a ficha pública completa de uma Obra, Para que eu possa conhecer seus dados autorais, editoriais e classificatórios.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar detalhes públicos de uma Obra

Scenario: Caminho Feliz - Consulta detalhada de Obra pública
Given que existe uma Obra com visibilidade Público
When eu acesso sua página de detalhes
Then o sistema deve retornar a ficha pública completa e as Edições públicas vinculadas

Scenario: Caminho Alternativo 1 - Enriquecimento para usuário autenticado (RN0050)
Given que eu possuo uma sessão stateful válida
When eu consulto os detalhes de uma Obra pública
Then o sistema pode incluir dados personalizados de posse e intenção de compra

Scenario: Caminho Alternativo 2 - Tentativa de acesso público a Obra privada (RN0046)
Given que a Obra solicitada possui visibilidade Privado
When eu tento acessá-la pela URL ou por seu identificador
Then o sistema deve recusar a consulta pública sem expor os dados da Obra
```

## RF0039: Consultar Edições públicas vinculadas a uma Obra

**História de usuário:** Como visitante ou usuário autenticado, Quero consultar as Edições públicas vinculadas a uma Obra, Para que eu possa comparar suas diferentes publicações físicas.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar Edições públicas vinculadas a uma Obra

Scenario: Caminho Feliz - Consulta das prévias de Edições
Given que uma Obra pública possui Edições públicas vinculadas
When eu consulto a seção de Edições da Obra
Then o sistema deve apresentar Capa, Número cronológico, Editora Brasileira, Tipo, Status e Total calculado de Volumes
And deve exibir uma amostragem dos Volumes públicos iniciais de cada Edição

Scenario: Caminho Alternativo 1 - Omissão de dependências privadas (RN0046)
Given que a Obra possui Edições ou Volumes privados
When eu consulto suas Edições públicas
Then o sistema deve omitir as Edições privadas e os Volumes privados da resposta
```

## RF0040: Consultar detalhes públicos de uma Edição

**História de usuário:** Como visitante ou usuário autenticado, Quero visualizar a ficha pública completa de uma Edição, Para que eu possa conhecer suas características e todos os Volumes públicos vinculados.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar detalhes públicos de uma Edição

Scenario: Caminho Feliz - Consulta detalhada de Edição pública
Given que existe uma Edição pública vinculada a uma Obra pública
When eu acesso a página de detalhes da Edição
Then o sistema deve retornar seus dados editoriais e a listagem paginada de Volumes públicos
And deve calcular o Total de Volumes a partir dos vínculos existentes no banco de dados

Scenario: Caminho Alternativo 1 - Navegação contextual
Given que eu estou nos detalhes de uma Edição pública
When eu seleciono a Obra matriz ou um Volume público
Then o sistema deve permitir a navegação para o respectivo detalhe público
```

## RF0041: Consultar detalhes públicos de um Volume

**História de usuário:** Como visitante ou usuário autenticado, Quero visualizar os detalhes de um Volume público, Para que eu possa consultar sua capa, publicação, identificadores, preço e sinopse.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar detalhes públicos de um Volume

Scenario: Caminho Feliz - Consulta detalhada de Volume público
Given que o Volume, sua Edição e sua Obra matriz estão públicos
When eu acesso a página de detalhes do Volume
Then o sistema deve retornar os dados públicos mais recentes do Volume
And deve apresentar a data conforme a precisão disponível

Scenario: Caminho Alternativo 1 - Acesso a Volume indisponível publicamente (RN0046)
Given que o Volume ou algum nível superior da hierarquia está privado
When eu tento acessar diretamente o detalhe do Volume
Then o sistema deve recusar a consulta pública sem expor seus dados
```

## RF0042: Consultar calendário público de lançamentos

**História de usuário:** Como visitante ou usuário autenticado, Quero consultar os Volumes previstos para determinado mês e ano, Para que eu possa acompanhar os próximos lançamentos do mercado brasileiro.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar calendário público de lançamentos

Scenario: Caminho Feliz - Consulta por mês e ano
Given que existem Volumes públicos com data de publicação compatível com o período informado
When eu consulto o calendário informando mês e ano
Then o sistema deve retornar os Volumes públicos previstos para esse período

Scenario: Caminho Alternativo 1 - Período padrão do servidor (RN0052)
Given que eu não informei mês nem ano
When eu acesso o calendário de lançamentos
Then o sistema deve utilizar automaticamente o mês e o ano vigentes no servidor
And não deve atribuir a um mês específico Volumes cuja data possua somente o ano

Scenario: Caminho Alternativo 2 - Busca e filtro editorial
Given que eu estou no calendário de lançamentos
When eu pesquiso por Título ou Autor e filtro por Editora Brasileira
Then o sistema deve retornar somente os lançamentos que atendam aos critérios combinados
```

## RF0043: Registrar posse individual de Volume

**História de usuário:** Como usuário autenticado, Quero marcar um Volume público como adquirido, Para que eu possa registrar esse item em minha Estante Digital.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Registrar posse individual de Volume

Scenario: Caminho Feliz - Adição à Estante Digital
Given que possuo uma sessão stateful válida e o Volume está público
When eu marco o Volume como adquirido
Then o sistema deve criar um único vínculo entre minha conta e o Volume
And deve iniciar o estado de leitura desse vínculo como "não lido"
And deve remover automaticamente o mesmo Volume da minha Lista de Desejos, caso exista

Scenario: Caminho Alternativo 1 - Volume indisponível para acervo pessoal (RN0056)
Given que o Volume, sua Edição ou sua Obra matriz está privado
When eu tento adicioná-lo à Estante Digital
Then o sistema deve recusar a operação sem criar o vínculo
```

## RF0044: Atualizar estado de leitura ou remover posse individual de Volume

**História de usuário:** Como usuário autenticado, Quero marcar como lido ou não lido um Volume que possuo e remover um Volume de minha Estante Digital, Para que eu possa manter o acervo e o acompanhamento da leitura atualizados.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Atualizar estado de leitura ou remover posse individual de Volume

Scenario: Caminho Feliz - Remoção do vínculo de posse
Given que possuo uma sessão stateful válida e o Volume está em minha Estante Digital
When eu solicito a remoção da posse
Then o sistema deve excluir somente o vínculo entre minha conta e o Volume
And deve excluir junto o estado de leitura associado ao vínculo
And não deve excluir o Volume do catálogo

Scenario: Caminho Feliz - Marcação de Volume como lido
Given que possuo uma sessão stateful válida e o Volume está em minha Estante Digital
When marco o Volume como lido
Then o sistema deve atualizar o estado de leitura somente do vínculo entre minha conta e esse Volume
And não deve criar uma lista ou entidade de leitura independente
```

## RF0045: Sincronizar registros de posse em lote

**História de usuário:** Como usuário autenticado, Quero adicionar e remover múltiplos Volumes de uma Edição em uma única operação, Para que eu possa atualizar minha coleção com rapidez e consistência.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Sincronizar registros de posse em lote

Scenario: Caminho Feliz - Sincronização transacional em lote
Given que possuo uma sessão stateful válida e estou visualizando uma Edição pública
When eu envio uma seleção com Volumes a adicionar e remover
Then o sistema deve aplicar toda a atualização em uma única transação
And deve remover da Lista de Desejos os Volumes adicionados à Estante Digital
And deve iniciar como "não lidos" os vínculos criados no lote
And deve excluir o estado de leitura dos vínculos removidos no lote

Scenario: Caminho Alternativo 1 - Falha durante a sincronização
Given que ao menos um item do lote viola uma regra de posse ou visibilidade
When o sistema processa a sincronização
Then deve cancelar toda a transação e preservar o estado anterior do acervo
```

## RF0046: Consultar registros da Estante Digital

**História de usuário:** Como usuário autenticado, Quero consultar os Volumes registrados em minha Estante Digital, Para que eu possa acompanhar minha coleção agrupada por Obra e Edição.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar registros da Estante Digital

Scenario: Caminho Feliz - Consulta agrupada da Estante Digital
Given que possuo uma sessão stateful válida e Volumes públicos em minha Estante
When eu acesso a Estante Digital
Then o sistema deve agrupar os Volumes por Obra e Edição
And deve apresentar a progressão em relação aos Volumes públicos cadastrados em cada Edição
And deve retornar o estado "lido" ou "não lido" de cada Volume vinculado

Scenario: Caminho Alternativo 1 - Omissão temporária de registros privados (RN0056)
Given que um Volume da minha Estante deixou de estar público
When eu consulto meu acervo
Then o sistema deve omitir o vínculo da resposta sem apagá-lo automaticamente
```

## RF0047: Registrar intenção de compra

**História de usuário:** Como usuário autenticado, Quero adicionar um Volume público à minha Lista de Desejos, Para que eu possa acompanhar os itens que pretendo adquirir.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Registrar intenção de compra

Scenario: Caminho Feliz - Adição à Lista de Desejos
Given que possuo uma sessão stateful válida e ainda não tenho o Volume na Estante Digital
When eu adiciono o Volume público à Lista de Desejos
Then o sistema deve criar um único vínculo de intenção de compra

Scenario: Caminho Alternativo 1 - Conflito por posse ativa (RN0057)
Given que o Volume já está registrado em minha Estante Digital
When eu tento adicioná-lo à Lista de Desejos
Then o sistema deve recusar a operação com HTTP 409
And deve informar que o Volume já pertence à minha Estante Digital
```

## RF0048: Remover intenção de compra

**História de usuário:** Como usuário autenticado, Quero remover um Volume de minha Lista de Desejos, Para que eu possa manter atualizadas minhas intenções de compra.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Remover intenção de compra

Scenario: Caminho Feliz - Remoção da Lista de Desejos
Given que possuo uma sessão stateful válida e o Volume está em minha Lista de Desejos
When eu solicito sua remoção
Then o sistema deve excluir somente o vínculo entre minha conta e o Volume
And não deve excluir o Volume do catálogo
```

## RF0049: Consultar Lista de Desejos

**História de usuário:** Como usuário autenticado, Quero consultar os Volumes registrados em minha Lista de Desejos, Para que eu possa acompanhar os itens que ainda pretendo adquirir.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Consultar Lista de Desejos

Scenario: Caminho Feliz - Consulta da Lista de Desejos
Given que possuo uma sessão stateful válida e Volumes públicos em minha Lista de Desejos
When eu acesso a Lista de Desejos
Then o sistema deve retornar os Volumes desejados com seus dados públicos essenciais

Scenario: Caminho Alternativo 1 - Omissão de item que se tornou privado (RN0056)
Given que um Volume desejado ou algum nível superior da hierarquia se tornou privado
When eu consulto a Lista de Desejos
Then o sistema deve omitir o item enquanto ele permanecer indisponível publicamente
```

## RF0050: Importar e gerenciar capa interna por URL

**História de usuário:** Como Administrador, Quero informar a URL de uma imagem para que o CoMangá a importe e armazene internamente, Para que eu possa gerenciar capas sem baixar arquivos nem acessar diretamente o provedor de mídia.

### Critérios de Aceite (Gherkin):

```gherkin
Feature: Importar e gerenciar capa interna por URL

Scenario: Caminho Feliz - Importação válida de capa
Given que possuo uma sessão ativa com privilégio de Administrador
And informo uma URL HTTPS que aponta para uma imagem válida
When solicito a importação da capa para uma Obra, Edição ou Volume
Then o sistema deve baixar, validar, processar e armazenar a imagem internamente
And deve associar à entidade somente a referência interna resultante
And não deve utilizar a URL externa como endereço de exibição (RN0058)

Scenario: Caminho Alternativo 1 - Origem inválida ou insegura (RN0059)
Given que a URL alcança uma rede proibida, excede limites ou não contém uma imagem válida
When solicito a importação da capa
Then o sistema deve rejeitar a operação com erro seguro e padronizado
And não deve criar associação nem objeto definitivo

Scenario: Caminho Alternativo 2 - Falha durante a substituição (RN0060)
Given que a entidade já possui uma capa interna
When ocorre uma falha ao importar ou associar a nova imagem
Then o sistema deve preservar a capa anterior
And deve permitir a limpeza segura dos recursos incompletos

Scenario: Caminho Alternativo 3 - Recusa de remoção sem substituição válida (RN0061)
Given que a entidade possui uma capa interna associada
When solicito a remoção da capa sem informar uma substituta interna válida
Then o sistema deve recusar a operação
And deve preservar a capa interna já associada, sem recorrer à URL externa de procedência
```
