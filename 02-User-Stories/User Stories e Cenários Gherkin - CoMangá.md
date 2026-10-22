# Histórias de usuário e critérios de aceite — CoMangá

Referência: RF01–RF27 e RN01–RN31 do SERS, versão 0.9 — homologação. Cada história preserva o ID do requisito correspondente.

## RF01 — Cadastrar conta

**Como** visitante, **quero** cadastrar conta **para** acessar recursos pessoais.

**Prioridade:** Alta. **Regras relacionadas:** RN01, RN02, RN03, RN08.

**Critério de referência:** Dados válidos criam conta pendente e iniciam o envio do link de ativação; duplicidade ou dados inválidos são recusados.

```gherkin
Funcionalidade: Cadastrar conta
  Cenário: Operação válida
    Dado que sou visitante e atendo às condições de acesso
    Quando cadastro dados válidos de nome, e-mail, nascimento e senha
    Então a conta permanece pendente e o link de ativação é enviado

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando nome, e-mail ou senha não atende às regras
    Então o cadastro é recusado sem criar conta
```

## RF02 — Ativar conta

**Como** titular da conta, **quero** ativar conta **para** acessar minha conta.

**Prioridade:** Alta. **Regras relacionadas:** RN03.

**Critério de referência:** Token válido ativa a conta uma vez; token expirado ou reutilizado não a ativa.

```gherkin
Funcionalidade: Ativar conta
  Cenário: Operação válida
    Dado que sou titular da conta e atendo às condições de acesso
    Quando abro um link de ativação válido
    Então a conta é ativada uma única vez

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando uso token expirado ou já consumido
    Então a conta não é ativada
```

## RF03 — Reenviar ativação

**Como** titular de uma conta pendente, **quero** reenviar ativação **para** concluir minha ativação.

**Prioridade:** Média. **Regras relacionadas:** RN03.

**Critério de referência:** Um novo link é enviado para conta elegível e o anterior perde a validade.

```gherkin
Funcionalidade: Reenviar ativação
  Cenário: Operação válida
    Dado que sou titular de uma conta pendente e atendo às condições de acesso
    Quando solicito novo link para minha conta pendente
    Então o novo link substitui o anterior

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando solicito reenvio para conta já ativa
    Então nenhum novo link de ativação é emitido
```

## RF04 — Iniciar sessão

**Como** titular de uma conta ativa, **quero** iniciar sessão **para** usar os recursos da conta.

**Prioridade:** Alta. **Regras relacionadas:** RN04.

**Critério de referência:** Credenciais válidas criam sessão persistida e cookie `HttpOnly`; credenciais inválidas não criam sessão.

```gherkin
Funcionalidade: Iniciar sessão
  Cenário: Operação válida
    Dado que sou titular de uma conta ativa e atendo às condições de acesso
    Quando informo credenciais válidas de conta ativa
    Então uma sessão persistida é criada e recebo cookie HttpOnly

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando informo credenciais inválidas ou conta pendente
    Então não recebo sessão válida
```

## RF05 — Encerrar sessão

**Como** pessoa autenticada, **quero** encerrar sessão **para** encerrar meu acesso.

**Prioridade:** Alta. **Regras relacionadas:** RN04.

**Critério de referência:** O cookie é removido e a sessão revogada deixa de autorizar rotas protegidas.

```gherkin
Funcionalidade: Encerrar sessão
  Cenário: Operação válida
    Dado que sou pessoa autenticada e atendo às condições de acesso
    Quando encerro minha sessão
    Então o cookie é removido e a sessão deixa de autorizar acesso

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando reutilizo o cookie da sessão encerrada
    Então o acesso protegido é recusado
```

## RF06 — Recuperar ou alterar senha

**Como** titular da conta, **quero** recuperar ou alterar senha **para** recuperar o acesso à conta.

**Prioridade:** Média. **Regras relacionadas:** RN02, RN04, RN05.

**Critério de referência:** Link válido permite definir nova senha; alteração em sessão exige a senha atual; a senha anterior deixa de autenticar.

```gherkin
Funcionalidade: Recuperar ou alterar senha
  Cenário: Operação válida
    Dado que sou titular da conta e atendo às condições de acesso
    Quando redefino a senha com token válido ou a altero informando a senha atual
    Então a nova senha passa a autenticar

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando uso token inválido ou senha atual incorreta
    Então a alteração é recusada
```

## RF07 — Consultar e atualizar perfil

**Como** pessoa autenticada, **quero** consultar e atualizar perfil **para** manter meus dados corretos.

**Prioridade:** Média. **Regras relacionadas:** RN01, RN06.

**Critério de referência:** A consulta retorna somente dados da própria conta; alteração válida aparece na consulta seguinte.

```gherkin
Funcionalidade: Consultar e atualizar perfil
  Cenário: Operação válida
    Dado que sou pessoa autenticada e atendo às condições de acesso
    Quando consulto e altero meu nome de usuário com dados válidos
    Então a consulta seguinte reflete o novo nome

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando tento consultar dados de outra conta
    Então os dados da outra conta não são exibidos
```

## RF08 — Selecionar perfil ativo

**Como** pessoa com múltiplos perfis, **quero** selecionar perfil ativo **para** usar as permissões adequadas.

**Prioridade:** Alta. **Regras relacionadas:** RN06.

**Critério de referência:** A troca atualiza o perfil ativo; menus e permissões passam a refletir o perfil escolhido, inclusive em outras abas após sincronização.

```gherkin
Funcionalidade: Selecionar perfil ativo
  Cenário: Operação válida
    Dado que sou pessoa com múltiplos perfis e atendo às condições de acesso
    Quando seleciono um perfil que me foi atribuído
    Então menus e permissões refletem o perfil ativo, inclusive nas demais abas

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando tento usar permissão administrativa com perfil padrão ativo
    Então a ação administrativa é recusada
```

## RF09 — Controlar conteúdo adulto

**Como** titular da conta, **quero** controlar conteúdo adulto **para** escolher o conteúdo que posso ver.

**Prioridade:** Alta. **Regras relacionadas:** RN06, RN08.

**Critério de referência:** Maior de 18 anos pode ativá-la; menor de idade não pode; o catálogo responde conforme a preferência efetiva.

```gherkin
Funcionalidade: Controlar conteúdo adulto
  Cenário: Operação válida
    Dado que sou titular da conta e atendo às condições de acesso
    Quando sou maior de idade e ativo a preferência adulta
    Então o catálogo passa a considerar a preferência

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando sou menor de idade e tento ativá-la
    Então a preferência é recusada
```

## RF10 — Excluir conta

**Como** titular da conta, **quero** excluir conta **para** encerrar meu vínculo com o serviço.

**Prioridade:** Média. **Regras relacionadas:** RN04, RN06.

**Critério de referência:** A conta, sessões e vínculos pessoais são removidos; outra conta não é afetada.

```gherkin
Funcionalidade: Excluir conta
  Cenário: Operação válida
    Dado que sou titular da conta e atendo às condições de acesso
    Quando confirmo a exclusão com minha senha atual
    Então minha conta, sessões e vínculos pessoais são excluídos

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando tento excluir outra conta
    Então a operação é recusada
```

## RF11 — Administrar usuários

**Como** administrador, **quero** administrar usuários **para** controlar o acesso administrativo.

**Prioridade:** Média. **Regras relacionadas:** RN06, RN07.

**Critério de referência:** Busca e filtros retornam os perfis atribuídos; alteração autorizada muda o acesso da conta alvo.

```gherkin
Funcionalidade: Administrar usuários
  Cenário: Operação válida
    Dado que sou administrador e atendo às condições de acesso
    Quando consulto contas e altero a atribuição de outra conta
    Então a atribuição autorizada passa a controlar o acesso

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando tento remover meu próprio perfil administrativo ou o último administrador
    Então a alteração é recusada
```

## RF12 — Manter opções administrativas

**Como** administrador, **quero** manter opções administrativas **para** manter os valores usados nos formulários.

**Prioridade:** Alta. **Regras relacionadas:** RN09, RN16.

**Critério de referência:** Novo valor aparece no formulário correspondente; valor inválido, duplicado ou em uso é recusado conforme a operação.

```gherkin
Funcionalidade: Manter opções administrativas
  Cenário: Operação válida
    Dado que sou administrador e atendo às condições de acesso
    Quando cadastro ou altero opção válida
    Então a opção aparece nos formulários correspondentes

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando tento duplicar valor ou excluir opção em uso
    Então a operação é recusada
```

## RF13 — Importar capas

**Como** administrador, **quero** importar capas **para** exibir capas internas válidas.

**Prioridade:** Alta. **Regras relacionadas:** RN23, RN24.

**Critério de referência:** A imagem é validada e armazenada na infraestrutura de mídia; falha não substitui uma capa já associada.

```gherkin
Funcionalidade: Importar capas
  Cenário: Operação válida
    Dado que sou administrador e atendo às condições de acesso
    Quando importo uma imagem válida por URL HTTPS
    Então a capa processada é armazenada internamente

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando a imagem falha na validação ou gravação
    Então a capa anteriormente associada permanece
```

## RF14 — Manter obras

**Como** administrador, **quero** manter obras **para** manter o catálogo correto.

**Prioridade:** Alta. **Regras relacionadas:** RN09, RN10, RN15, RN16, RN23, RN24, RN25, RN26, RN27.

**Critério de referência:** Uma obra válida é recuperável após o cadastro; alterações preservam campos não enviados; exclusão respeita vínculos e visibilidade.

```gherkin
Funcionalidade: Manter obras
  Cenário: Operação válida
    Dado que sou administrador e atendo às condições de acesso
    Quando mantenho uma obra com os campos mínimos e seus vínculos
    Então a obra é recuperável e mudanças parciais preservam demais campos

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando tento duplicar título ou excluir obra protegida
    Então a operação é recusada
```

## RF15 — Manter edições

**Como** administrador, **quero** manter edições **para** representar as publicações brasileiras.

**Prioridade:** Alta. **Regras relacionadas:** RN11, RN15, RN16, RN28, RN30.

**Critério de referência:** Cada edição tem número único dentro da obra; acabamento, formato e miolos aceitam os domínios válidos; a capa exibida deriva do volume 1.

```gherkin
Funcionalidade: Manter edições
  Cenário: Operação válida
    Dado que sou administrador e atendo às condições de acesso
    Quando mantenho uma edição válida dentro da obra
    Então o número é único na obra e a capa exibida deriva do volume 1

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando tento repetir número ou publicar sem volume 1 com capa
    Então a operação é recusada
```

## RF16 — Manter volumes

**Como** administrador, **quero** manter volumes **para** registrar os livros de cada edição.

**Prioridade:** Alta. **Regras relacionadas:** RN12, RN13, RN14, RN15, RN16, RN23, RN24, RN29.

**Critério de referência:** O volume válido aparece na edição com número, capa e dados informados; número repetido na mesma edição é recusado.

```gherkin
Funcionalidade: Manter volumes
  Cenário: Operação válida
    Dado que sou administrador e atendo às condições de acesso
    Quando mantenho um volume válido dentro da edição
    Então o volume aparece com seu número e capa

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando tento repetir número na edição
    Então a operação é recusada
```

## RF17 — Controlar visibilidade

**Como** administrador, **quero** controlar visibilidade **para** controlar o que aparece no catálogo público.

**Prioridade:** Alta. **Regras relacionadas:** RN10, RN11, RN14.

**Critério de referência:** Catálogo público passa a refletir a mudança; a visibilidade dos volumes acompanha a edição na mesma operação.

```gherkin
Funcionalidade: Controlar visibilidade
  Cenário: Operação válida
    Dado que sou administrador e atendo às condições de acesso
    Quando publico obra e edição válidas
    Então o catálogo reflete a publicação e os volumes acompanham a edição

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando tento publicar edição de obra privada
    Então a operação é recusada
```

## RF18 — Explorar e pesquisar catálogo público

**Como** visitante ou pessoa autenticada, **quero** explorar e pesquisar catálogo público **para** encontrar títulos do meu interesse.

**Prioridade:** Alta. **Regras relacionadas:** RN08, RN17, RN18, RN27, RN30.

**Critério de referência:** As duas listagens abrem sem termo; os resultados contêm somente registros visíveis e compatíveis com os filtros, e uma nova pesquisa produz o conjunto correspondente.

```gherkin
Funcionalidade: Explorar e pesquisar catálogo público
  Cenário: Operação válida
    Dado que sou visitante ou pessoa autenticada e atendo às condições de acesso
    Quando abro as listagens sem termo ou aplico texto, filtros e ordenação
    Então recebo apenas registros públicos compatíveis com os parâmetros

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando mudo os filtros ou a pesquisa
    Então o resultado corresponde à nova consulta
```

## RF19 — Navegar pelas fichas

**Como** visitante ou pessoa autenticada, **quero** navegar pelas fichas **para** compreender a relação entre as publicações.

**Prioridade:** Alta. **Regras relacionadas:** RN08, RN17, RN30.

**Critério de referência:** É possível seguir obra → edição → volume e voltar pelo caminho contextual; URLs de registros privados não expõem seus dados.

```gherkin
Funcionalidade: Navegar pelas fichas
  Cenário: Operação válida
    Dado que sou visitante ou pessoa autenticada e atendo às condições de acesso
    Quando abro uma obra e sigo para edição e volume
    Então vejo as fichas públicas e sua navegação contextual

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando acesso diretamente URL de registro privado
    Então os dados privados não são expostos
```

## RF20 — Consultar obras por autor

**Como** visitante ou pessoa autenticada, **quero** consultar obras por autor **para** descobrir outras obras do mesmo autor.

**Prioridade:** Média. **Regras relacionadas:** RN08, RN17.

**Critério de referência:** A página lista as obras públicas associadas ao autor e permite abrir suas fichas.

```gherkin
Funcionalidade: Consultar obras por autor
  Cenário: Operação válida
    Dado que sou visitante ou pessoa autenticada e atendo às condições de acesso
    Quando abro a página de um autor a partir de uma obra
    Então vejo suas obras públicas e consigo abrir as fichas

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando o autor também está vinculado a obra privada
    Então a obra privada não aparece
```

## RF21 — Consultar lançamentos

**Como** visitante ou pessoa autenticada, **quero** consultar lançamentos **para** acompanhar publicações do período.

**Prioridade:** Alta. **Regras relacionadas:** RN08, RN13, RN17, RN19.

**Critério de referência:** A lista usa o período selecionado e inclui apenas volumes com mês conhecido e visibilidade pública.

```gherkin
Funcionalidade: Consultar lançamentos
  Cenário: Operação válida
    Dado que sou visitante ou pessoa autenticada e atendo às condições de acesso
    Quando seleciono mês e ano de lançamentos
    Então vejo volumes públicos com mês de lançamento conhecido

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando o volume possui apenas o ano da data
    Então ele não é atribuído a mês arbitrário
```

## RF22 — Manter coleção individual

**Como** pessoa autenticada, **quero** manter coleção individual **para** registrar meus volumes e leituras.

**Prioridade:** Alta. **Regras relacionadas:** RN20, RN21.

**Critério de referência:** A alteração aparece na ficha do volume e nas telas da coleção; remover um volume da coleção também remove seu estado de leitura.

```gherkin
Funcionalidade: Manter coleção individual
  Cenário: Operação válida
    Dado que sou pessoa autenticada e atendo às condições de acesso
    Quando adiciono ou removo volume da coleção e altero sua leitura
    Então a ficha e a coleção refletem o novo estado

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando removo volume lido da coleção
    Então o estado de leitura também é removido
```

## RF23 — Alterar coleção em lote

**Como** pessoa autenticada, **quero** alterar coleção em lote **para** atualizar vários volumes de uma vez.

**Prioridade:** Alta. **Regras relacionadas:** RN20, RN21, RN22, RN31.

**Critério de referência:** O lote é aplicado integralmente ou não altera nenhum vínculo; erro de validação ou gravação não deixa mudanças parciais.

```gherkin
Funcionalidade: Alterar coleção em lote
  Cenário: Operação válida
    Dado que sou pessoa autenticada e atendo às condições de acesso
    Quando salvo a seleção de volumes da edição na coleção
    Então todas as mudanças do lote são aplicadas juntas

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando um volume do lote falha na validação ou gravação
    Então nenhuma mudança do lote é persistida
```

## RF24 — Manter lista de desejos

**Como** pessoa autenticada, **quero** manter lista de desejos **para** organizar os volumes que desejo.

**Prioridade:** Alta. **Regras relacionadas:** RN20, RN21.

**Critério de referência:** A lista reflete a alteração; um volume que está na coleção não pode ser adicionado à lista de desejos.

```gherkin
Funcionalidade: Manter lista de desejos
  Cenário: Operação válida
    Dado que sou pessoa autenticada e atendo às condições de acesso
    Quando adiciono ou removo volume da lista de desejos
    Então a lista reflete a alteração

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando tento desejar volume que já está na coleção
    Então a inclusão na lista é recusada
```

## RF25 — Consultar coleção e lista de desejos

**Como** pessoa autenticada, **quero** consultar coleção e lista de desejos **para** acompanhar meu acervo pessoal.

**Prioridade:** Alta. **Regras relacionadas:** RN20, RN21.

**Critério de referência:** Cada tela mostra somente os vínculos da própria conta e atualiza os totais após uma alteração.

```gherkin
Funcionalidade: Consultar coleção e lista de desejos
  Cenário: Operação válida
    Dado que sou pessoa autenticada e atendo às condições de acesso
    Quando pesquiso ou filtro minha coleção ou lista de desejos
    Então vejo somente meus vínculos e os totais correspondentes

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando outra conta consulta seus dados
    Então meus vínculos não aparecem para ela
```

## RF26 — Alterar lista de desejos em lote

**Como** pessoa autenticada, **quero** alterar lista de desejos em lote **para** atualizar vários desejos de uma vez.

**Prioridade:** Média. **Regras relacionadas:** RN20, RN21, RN31.

**Critério de referência:** O lote é aplicado integralmente ou não altera nenhum vínculo; erro de validação ou gravação não deixa mudanças parciais.

```gherkin
Funcionalidade: Alterar lista de desejos em lote
  Cenário: Operação válida
    Dado que sou pessoa autenticada e atendo às condições de acesso
    Quando salvo a seleção de volumes da edição na lista de desejos
    Então todas as mudanças do lote são aplicadas juntas

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando um volume do lote falha na validação ou gravação
    Então nenhuma mudança do lote é persistida
```

## RF27 — Alterar estado de leitura em lote

**Como** pessoa autenticada, **quero** alterar estado de leitura em lote **para** atualizar a leitura de vários volumes de uma vez.

**Prioridade:** Média. **Regras relacionadas:** RN20, RN21, RN31.

**Critério de referência:** O lote é aplicado integralmente ou não altera nenhum estado; volumes fora da coleção não recebem estado de leitura e erro não deixa mudanças parciais.

```gherkin
Funcionalidade: Alterar estado de leitura em lote
  Cenário: Operação válida
    Dado que sou pessoa autenticada e atendo às condições de acesso
    Quando marco em lote volumes da coleção como lidos ou não lidos
    Então todas as mudanças de leitura são aplicadas juntas

  Cenário: Restrição de domínio
    Dado que a operação está sujeita às regras de negócio
    Quando o lote inclui volume fora da coleção ou falha na gravação
    Então nenhum estado de leitura do lote é alterado
```
