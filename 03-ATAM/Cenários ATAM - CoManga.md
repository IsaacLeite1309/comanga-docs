# Cenários de qualidade e análise arquitetural — CoMangá

Versão 0.9 — homologação. Os cenários derivam dos RNFs vigentes no SERS. A medida descreve comportamento verificável da implementação; não se presume objetivo de desempenho ou recuperação sem medição.

## RNF01 — Paginação

**Fonte:** Cliente da API.

**Estímulo:** Cliente solicita página da listagem ou `limit=51`.

**Ambiente:** aplicação em execução, nas condições de acesso descritas no requisito.

**Artefato:** API com listagem paginada.

**Resposta e medida:** Retorna no máximo 50 itens por página e recusa `limit=51`; detalhes podem trazer seus próprios vínculos.

**Verificação:** Conferir os esquemas de entrada e a recusa de `limit=51` nas listagens paginadas.

**Origem:** RNF01 do [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md).

## RNF02 — Segurança de credenciais

**Fonte:** Titular da conta.

**Estímulo:** Conta é criada ou a senha é trocada.

**Ambiente:** aplicação em execução, nas condições de acesso descritas no requisito.

**Artefato:** Banco e resposta do perfil.

**Resposta e medida:** Persistência contém hash bcrypt com fator 10; a resposta não contém a senha.

**Verificação:** Inspecionar criação e troca de senha e o contrato de consulta da própria conta.

**Origem:** RNF02 do [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md).

## RNF03 — Segurança de sessão

**Fonte:** Titular da conta.

**Estímulo:** Sessão é criada em ambiente de produção.

**Ambiente:** aplicação em execução, nas condições de acesso descritas no requisito.

**Artefato:** Cookie e banco de sessões.

**Resposta e medida:** Cookie contém identificador opaco HttpOnly e Secure conforme configuração; banco contém apenas hash do token.

**Verificação:** Inspecionar a criação da sessão, o cabeçalho `Set-Cookie` e a configuração efetiva do ambiente.

**Origem:** RNF03 do [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md).

## RNF04 — Autorização

**Fonte:** Conta com múltiplos perfis.

**Estímulo:** Conta atribuída a administrador acessa rota com perfil padrão ativo.

**Ambiente:** aplicação em execução, nas condições de acesso descritas no requisito.

**Artefato:** Middleware de autorização.

**Resposta e medida:** A operação administrativa é negada até que o perfil administrador seja ativado.

**Verificação:** Conferir os middlewares das rotas e comparar respostas de uma mesma conta nos perfis padrão e administrador.

**Origem:** RNF04 do [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md).

## RNF05 — Proteção de login

**Fonte:** Cliente do login.

**Estímulo:** Mesmo IP envia mais de cinco logins inválidos em cinco minutos.

**Ambiente:** aplicação em execução, nas condições de acesso descritas no requisito.

**Artefato:** Processo da API.

**Resposta e medida:** Tentativas adicionais recebem HTTP 429 enquanto a janela estiver ativa; contador é local ao processo.

**Verificação:** Conferir o limitador de login e observar a resposta às tentativas consecutivas.

**Origem:** RNF05 do [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md).

## RNF06 — Integridade

**Fonte:** Administrador ou usuário do acervo.

**Estímulo:** Uma gravação falha após início de operação composta.

**Ambiente:** aplicação em execução, nas condições de acesso descritas no requisito.

**Artefato:** Prisma e PostgreSQL.

**Resposta e medida:** Transação desfaz alterações parciais; FKs, unicidade e CHECK impedem estados inválidos.

**Verificação:** Conferir as transações dessas operações e as restrições nas migrations.

**Origem:** RNF06 do [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md).

## RNF07 — Tratamento de erro

**Fonte:** Cliente da API.

**Estímulo:** Requisição causa exceção inesperada.

**Ambiente:** aplicação em execução, nas condições de acesso descritas no requisito.

**Artefato:** Tratador global da API.

**Resposta e medida:** Resposta JSON não contém stack trace; log estruturado registra a falha e dados da requisição.

**Verificação:** Conferir o middleware de erro, o formato da resposta e os campos do log.

**Origem:** RNF07 do [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md).

## RNF08 — Cache e atualização

**Fonte:** Pessoa navegando no catálogo.

**Estímulo:** Pessoa retorna a uma consulta ou altera acervo.

**Ambiente:** aplicação em execução, nas condições de acesso descritas no requisito.

**Artefato:** Cache em memória do navegador.

**Resposta e medida:** Consulta atual é reutilizada por até 2 minutos; entrada sem uso expira após 30 minutos; mutação atualiza ou invalida dados relacionados.

**Verificação:** Conferir a configuração do cliente de consultas e os pontos de invalidação ou atualização após mutações.

**Origem:** RNF08 do [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md).

## RNF09 — Responsividade

**Fonte:** Pessoa no navegador.

**Estímulo:** Largura da janela cruza 768 px.

**Ambiente:** aplicação em execução, nas condições de acesso descritas no requisito.

**Artefato:** Navegação do frontend.

**Resposta e medida:** Abaixo de 768 px aparece a barra inferior; a partir de 768 px aparece a navegação lateral.

**Verificação:** Conferir os pontos de quebra e os destinos oferecidos nas duas apresentações.

**Origem:** RNF09 do [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md).

## RNF10 — Capas e layout

**Fonte:** Serviço de mídia ou navegador.

**Estímulo:** Uma capa tem proporção diferente ou falha ao carregar.

**Ambiente:** aplicação em execução, nas condições de acesso descritas no requisito.

**Artefato:** Componente visual e armazenamento.

**Resposta e medida:** Área da capa mantém proporção 2:3 e mostra placeholder; banco contém somente chave e metadados do arquivo.

**Verificação:** Conferir o componente de capa, seu tratamento de erro e o esquema de mídia.

**Origem:** RNF10 do [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md).

## RNF11 — Estados de interface

**Fonte:** Pessoa usando as páginas públicas ou pessoais.

**Estímulo:** Consulta está carregando ou não encontra resultados.

**Ambiente:** aplicação em execução, nas condições de acesso descritas no requisito.

**Artefato:** Pesquisa, lançamentos, coleção e lista.

**Resposta e medida:** A interface apresenta estado de carregamento ou vazio correspondente, sem exibir conteúdo incorreto.

**Verificação:** Conferir os componentes dessas quatro páginas nos dois estados.

**Origem:** RNF11 do [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md).

## Decisões e limites

- O cache reduz novas esperas no navegador, mas o backend continua responsável pela autorização e pela visibilidade.
- O limitador de login mantém estado em memória de um processo; múltiplas instâncias exigiriam coordenação adicional para compartilhar o contador.
- Disponibilidade, percentis de latência e restauração de backup não receberam metas neste documento porque não foram medidos ou ensaiados para esta versão.
