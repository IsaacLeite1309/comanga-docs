# Cenários ATAM do CoMangá

Cenários de atributos de qualidade para os requisitos não funcionais (RNF01 a RNF16) do [SERS](../01-SERS/SERS%20-%20CoMang%C3%A1%20-%20Revisado.md). Cada cenário descreve fonte, estímulo, artefato, ambiente, resposta e medida, seguidos da evidência disponível. As decisões arquiteturais citadas estão no [DAS](../04-DAS/DAS%20-%20CoManga%20-%20Atualizado.md).

A linha **Situação** separa o que está garantido do que ainda é meta:

- **Verificado por teste**: a medida é conferida por teste automatizado da API ou da Web.
- **Implementado, sem teste da medida**: o mecanismo existe no código, mas nenhum teste confere o valor da medida.
- **Meta com medição pendente**: há ferramenta de medição, mas ainda não existe resultado registrado.
- **Meta sem mecanismo de medição**: não há teste nem ferramenta no projeto que produza a medida.
- **Parcial**: parte da medida é atendida e verificada; outra parte não.

As suítes de teste citadas estão em `comanga-api/__tests__/` e `comanga-web/src/`. Testes de integração da API exigem banco de teste (`npm run check:integration`); a validação `npm run check` é executada localmente. O GitHub Actions da conta está bloqueado por cobrança, portanto nenhum resultado de CI serve como evidência.

## Categoria: Desempenho

### RNF01 — Desempenho de tempo de resposta do motor de busca de Obras e Edições

- **Fonte:** vários clientes HTTP simultâneos (a Web ou scripts) consultando o catálogo público.
- **Estímulo:** requisições de pesquisa paginada com termo textual e filtros combinados.
- **Artefato:** `GET /api/public/works` e `GET /api/public/editions` (módulo `public-catalog`), Prisma e PostgreSQL.
- **Ambiente:** operação normal, banco populado, rede cliente-servidor estável com latência de até 100 ms.
- **Resposta:** a API valida os filtros, monta a consulta com as restrições de visibilidade e de conteúdo adulto e a executa sobre índices trigrama (`pg_trgm`) para busca parcial e índices compostos para visibilidade e classificações. Página e contagem são obtidas na mesma transação, com `skip`/`take` no banco.
- **Medida:** P95 do tempo de resposta menor ou igual a 500 ms no período aferido.
- **Evidência:** `publicCatalogQueryPlans.test.js` confere, no banco de teste, que as buscas usam os índices esperados. `npm run load:test` calcula o P95 por estágio e marca RNF01 como aprovado ou reprovado (`loadTest.unit.test.js` testa esse cálculo). O script mede o tempo no cliente, incluindo a rede; o resultado é, portanto, conservador frente à definição do SERS, que considera só o processamento do servidor. Nenhuma execução está registrada (`comanga-api/docs/operations/capacity-test.md`).
- **Situação:** meta com medição pendente.

### RNF02 — Capacidade de vazão e concorrência da API Pública

- **Fonte:** muitos usuários externos acessando o catálogo ao mesmo tempo (por exemplo, durante a apresentação do projeto).
- **Estímulo:** pico sustentado de 20 requisições por segundo nas rotas públicas de leitura.
- **Artefato:** rotas `/api/public/*`, processo único da API na Render, pool de conexões do Prisma (5 conexões por padrão, espera máxima de 10 s).
- **Ambiente:** produção ou homologação equivalente, banco populado, sem manutenção concorrente. O plano gratuito da Render pode estar adormecido; a partida a frio é registrada à parte.
- **Resposta:** requisições acima da capacidade do pool aguardam conexão; esgotada a espera, a falha vira resposta 5xx padronizada pelo tratador de erros, sem derrubar o processo. As rotas públicas não têm rate limiting.
- **Medida:** pelo menos 20 RPS sustentados com taxa de respostas HTTP 5xx inferior a 1%.
- **Evidência:** `npm run load:test` executa estágios de 10, 25, 50 e 100 usuários virtuais, calcula RPS e taxa de 5xx por estágio e, com `OPS_METRICS_TOKEN`, coleta CPU, memória e conexões do PostgreSQL em `/health/metrics`. Nenhuma execução está registrada.
- **Situação:** meta com medição pendente. Nenhuma capacidade deve ser afirmada antes de um relatório real.

### RNF03 — Eficiência de tráfego e limite de paginação

- **Fonte:** a Web ou um script chamando a API diretamente.
- **Estímulo:** listagem sem parâmetro de paginação ou com `limit` maior que 50.
- **Artefato:** esquemas de consulta do `public-catalog` e das listagens administrativas de Obras, Edições e Volumes.
- **Ambiente:** operação normal, banco com centenas ou milhares de registros.
- **Resposta:** sem `limit`, as listas públicas usam 12 itens por página. Com `limit` acima de 50, a API **recusa** a requisição com HTTP 400 ("Filtros de consulta inválidos." ou "Parâmetros de consulta inválidos."), sem consultar o banco; ela não trunca o valor. Os detalhes da Edição paginam seus Volumes; os detalhes da Obra listam todas as Edições públicas, com prévia de até 3 Volumes cada; `catalog-options` devolve todos os valores ativos, sem paginação. As listagens administrativas de contas (`admin/users/domain.ts`) e de valores de listas (`admin/options/schemas.ts`) aceitam até 100 itens (SERS 6.3).
- **Medida:** nenhuma resposta traz mais de 50 itens por página; o corpo JSON não ultrapassa 2 MB.
- **Evidência:** o limite de 50 é verificado por `publicCatalog.test.js` (limite 51 recebe 400), `publicCatalog.unit.test.js`, `publicAuthorWorks.unit.test.js`, `publicEditionDetails.unit.test.js` e `adminModules.unit.test.js`. O tamanho de 2 MB não é imposto pelo código: é consequência do limite de itens e só é medido pelo `load:test`, que reprova respostas acima de 2 MB ou com mais de 50 itens.
- **Situação:** parcial. Limite de 50 itens verificado por teste no catálogo público e nas listagens de Obras, Edições e Volumes; contas e valores de listas ficam fora dele (SERS 6.3); limite de 2 MB é meta com medição pendente.

### RNF04 — Desempenho de entrega de mídia estática (Capas)

- **Fonte:** navegadores exibindo a pesquisa ou os detalhes do catálogo.
- **Estímulo:** carregamento simultâneo de dezenas de capas.
- **Artefato:** Cloudflare R2 e a origem pública de mídia (`MEDIA_PUBLIC_BASE_URL`), metadados em `media_assets`/`media_variants`, componente `CatalogCover` da Web.
- **Ambiente:** operação normal em produção.
- **Resposta:** a API devolve apenas URLs HTTPS derivadas da chave interna e da configuração, preferindo a variante 640x960 em WebP. O navegador busca as imagens direto da origem de mídia, sem passar pela API nem pelo banco. Os objetos são gravados com `Cache-Control: public, max-age=31536000, immutable`; a Web usa carregamento tardio (`loading="lazy"`) fora da primeira dobra.
- **Medida:** 0 bytes de imagem ou base64 no PostgreSQL; URLs de capa sempre derivadas da configuração interna; tempo até o primeiro byte das capas de no máximo 300 ms no P90.
- **Evidência:** o schema só tem colunas de metadados e chaves; `internalCoverMediaSchema.unit.test.js` e `internalCoverMedia.test.js` conferem as constraints de mídia; `mediaPublicUrl.unit.test.js` confere a montagem das URLs e a exigência de HTTPS. Não há ferramenta no projeto que meça o TTFB das capas.
- **Situação:** parcial. Ausência de binários no banco e URLs internas verificadas por teste; o TTFB P90 de 300 ms é meta sem mecanismo de medição.

## Categoria: Segurança

### RNF05 — Armazenamento irreversível de credenciais (Hashing de Senhas)

- **Fonte:** atacante com acesso de leitura ao banco ou auditor de segurança.
- **Estímulo:** leitura direta da tabela `users` para extrair senhas.
- **Artefato:** coluna `users.password_hash` e os fluxos de cadastro, redefinição e alteração de senha.
- **Ambiente:** banco com contas ativas.
- **Resposta:** o banco contém apenas hashes bcrypt, com salt gerado a cada hash. Senhas acima de 72 bytes em UTF-8 são recusadas antes do bcrypt, para que nenhum trecho seja ignorado. Tokens de redefinição e de sessão também são guardados apenas como hash SHA-256.
- **Medida:** 100% das senhas armazenadas em bcrypt com custo mínimo 10; nenhum registro em texto plano, MD5, SHA-1 ou criptografia reversível.
- **Evidência:** cadastro, redefinição e alteração de senha chamam `bcrypt.hash(senha, 10)`. `authRegister.test.js` confere que o valor gravado difere da senha e é validado por `bcrypt.compare`; `authRules.unit.test.js` confere o limite de 72 bytes. Nenhum teste confere explicitamente o custo 10 no hash gravado.
- **Situação:** parcial. O uso de bcrypt é verificado por teste; o custo 10, só por inspeção do código.

### RNF06 — Gerenciamento de sessão stateful e transporte seguro

- **Fonte:** script malicioso injetado no navegador; o próprio usuário ou o sistema encerrando a sessão.
- **Estímulo:** tentativa de ler o identificador de sessão por JavaScript e, depois, uso de um cookie de sessão já revogada.
- **Artefato:** cookie de sessão, tabela `sessions`, `authMiddleware` e as operações que revogam sessões.
- **Ambiente:** produção, usuário autenticado sob HTTPS, Web e API na mesma origem para o navegador (a Vercel reescreve `/api`).
- **Resposta:** o cookie é HttpOnly, então o script não o lê. Cada requisição protegida calcula o hash do cookie e exige sessão não revogada e conta Ativada. Cookie ausente, sessão inexistente ou revogada recebem 401 com "Sua sessão é inválida ou foi encerrada. Por favor, faça login novamente.". Uma sessão encontrada cuja conta não está Ativada recebe 403 com "Sua conta nao esta ativa para acessar este recurso." (texto atual da API, sem acentos). A revogação ocorre no logout (sessão atual), na redefinição de senha (todas as sessões), na alteração de senha (todas menos a atual) e na exclusão da conta (remoção em cascata). Quando outro Administrador remove o perfil Administrador da conta, as sessões com esse perfil ativo passam a Usuário Padrão e perdem o acesso administrativo na requisição seguinte. O CORS aceita credenciais só das origens de `CORS_ORIGIN`.
- **Medida:** cookie com HttpOnly e SameSite=Strict (padrão), e Secure em produção; 0% de exposição do identificador ao JavaScript; 100% das requisições com sessão revogada recusadas com 401.
- **Evidência:** `authLoginSession.test.js` confere HttpOnly e SameSite=Strict no cookie e o bloqueio após revogação; `authLogout.test.js` confere que o cookie antigo é recusado após logout; `sessionSecurityRegressions.unit.test.js` confere a busca pelo hash e o descarte de sessões revogadas; `passwordRecovery.unit.test.js` e `userAccountSettings.unit.test.js` conferem as revogações na redefinição e na alteração de senha; `middlewares.unit.test.js` confere o rebaixamento do perfil ativo quando a atribuição some; `app.unit.test.js` confere o CORS. O atributo Secure depende de `NODE_ENV=production`, de `x-forwarded-proto: https` ou de `COOKIE_SECURE`, e não tem teste. Não há expiração da sessão por tempo; ela vale até ser revogada.
- **Situação:** verificado por teste, exceto o atributo Secure (implementado, sem teste). A sessão não expira por tempo; a expiração é planejada no RNF17.

### RNF07 — Controle de acesso por perfil ativo via middleware

O controle é feito por **perfil ativo**, não por uma role única: a rota exige que o perfil ativo da sessão seja Administrador e que a conta ainda tenha esse perfil concedido.

- **Fonte:** conta autenticada sem permissão administrativa efetiva: perfil ativo Usuário Padrão (mesmo que a conta tenha o perfil Administrador concedido), ou perfil ativo Administrador cuja atribuição foi removida.
- **Estímulo:** chamada a uma rota `/api/admin/*`, por exemplo excluir uma Obra ou alterar o perfil de outra conta.
- **Artefato:** `authMiddleware` e `requireRole('Administrador')` em todas as rotas administrativas.
- **Ambiente:** operação normal, sessão stateful válida.
- **Resposta:** o middleware de sessão carrega a conta, os perfis concedidos e o perfil ativo; `requireRole` recusa antes de chamar o controlador. Nenhuma regra de negócio nem transação do catálogo é executada. Sem sessão, a resposta é 401; conta não Ativada recebe 403. Na Web, `ProtectedRoute` envia ao perfil quem não tem o perfil ativo Administrador, mas a proteção efetiva é a da API.
- **Medida:** HTTP 403 em 100% das rotas administrativas para quem não tem o perfil ativo Administrador vigente; nenhuma regra de negócio executada. As únicas operações de banco antes da recusa são a leitura da sessão e, no máximo a cada 5 minutos, a atualização do último uso.
- **Evidência:** `adminRoutesAuthorization.test.js` percorre todas as rotas administrativas declaradas e confere 401 sem sessão, 401 com sessão revogada, 403 com sessão de Usuário Padrão e 403 com conta administrativa não ativada; `middlewares.unit.test.js` confere que perfil ativo sem atribuição e atribuição sem perfil ativo são recusados; `adminRouteProtection.test.tsx` e `ProtectedRoute.test.tsx` conferem as guardas da Web.
- **Situação:** verificado por teste.

### RNF08 — Limitação de Taxa de Autenticação (Rate Limiting)

- **Fonte:** script de força bruta ou credential stuffing a partir de um único IP.
- **Estímulo:** mais de 5 tentativas de login com credenciais inválidas em 5 minutos.
- **Artefato:** limitador de login (`loginRateLimiter`) com armazenamento em memória (`MemoryRateLimitStore`), antes do controlador de login.
- **Ambiente:** produção, rota de login exposta à internet.
- **Resposta:** o limitador conta, por IP, as respostas 401 do login. Com 5 falhas registradas, a requisição seguinte recebe 429 (`LOGIN_RATE_LIMITED`) antes do controlador, sem consulta ao banco. A janela é fixa: dura 5 minutos a partir da primeira falha e não se renova a cada tentativa. Um login bem-sucedido zera a contagem. Recuperação e redefinição de senha têm limite próprio (5 requisições por IP em 15 minutos, 429 com `Retry-After`), e a alteração de senha autenticada tem limite por conta (5 falhas em 5 minutos).
- **Medida:** a 6ª tentativa após 5 falhas no mesmo IP recebe 429, sem consulta ao banco, até 5 minutos após a primeira falha.
- **Evidência:** `middlewares.unit.test.js` confere o bloqueio na 6ª tentativa e o reinício após sucesso; `authLoginSession.test.js` confere o 429 após cinco falhas na API real; `infrastructureContracts.unit.test.js`, `passwordRecovery.unit.test.js` e `userAccountSettings.test.js` conferem os demais limitadores.
- **Limitações:** a contagem fica na memória do processo, reinicia quando a API reinicia e não seria compartilhada entre várias instâncias. O IP vem de `req.ip` com um salto de proxy confiável (`trust proxy = 1`); com a Vercel reescrevendo `/api` para a Render, é preciso confirmar em produção que esse valor é o IP do cliente, e não o do proxy. O limitador de recuperação passa a recusar qualquer IP novo quando o mapa em memória chega a 10.000 IPs. A emissão de token de redefinição é limitada a 1 por conta a cada 60 segundos. Cadastro e reenvio de ativação não têm rate limiting, e o reenvio responde 404 para e-mail não cadastrado, o que permite enumerar contas; essas rotas ficam fora da medida deste cenário.
- **Situação:** verificado por teste para o login, com as limitações acima.

### Cenário complementar — Importação remota de capas

Cenário de segurança ligado ao RF0050 e ao RN0059, sem RNF próprio no SERS.

- **Fonte:** Administrador autenticado, de boa-fé ou com a conta comprometida.
- **Estímulo:** importação de capa a partir de uma URL que aponta para a rede interna, redireciona para ela ou entrega um arquivo grande, lento ou que não é imagem.
- **Artefato:** `POST /api/admin/media/covers`, `RemoteImageDownloader`, `CoverImageProcessor` e `R2MediaStorage`.
- **Ambiente:** produção, credenciais do R2 configuradas.
- **Resposta:** a rota exige sessão com perfil ativo Administrador. A importação é genérica, sem regra por provedor: aceita só HTTPS sem credenciais; resolve o DNS e recusa `localhost` e endereços privados, reservados ou locais; fixa o IP resolvido na conexão; segue até 3 redirecionamentos, revalidando cada um; aceita só JPEG, PNG, WebP ou AVIF; interrompe o download acima de 10 MiB ou após 10 s por requisição; recusa imagens acima de 40 milhões de pixels. Só depois grava as variantes WebP no R2. Falhas não criam associação e provocam uma tentativa de remover os objetos já enviados; a capa anterior de uma Obra ou Volume permanece. A remoção compensatória pode falhar, deixando objetos órfãos no R2.
- **Medida:** 0 requisições a destinos privados ou reservados; 0 objetos órfãos no R2 após falha; limites de tamanho, tempo e pixels respeitados.
- **Evidência:** `remoteImageDownloader.unit.test.js`, `coverImageProcessor.unit.test.js`, `coverImportService.unit.test.js`, `failureIsolationRegressions.unit.test.js`, `r2MediaStorage.unit.test.js` e `app.unit.test.js` (rota protegida).
- **Limitações:** a checagem de endereços não reconhece prefixos IPv6 que embutem IPv4 (`64:ff9b::/96` e `2002::/16`), o que deixa parte da meta "0 destinos privados" sem atendimento. Se a persistência no banco falhar e a remoção compensatória no R2 também falhar, `CoverImportService` ignora o erro da remoção e propaga a falha original. Os objetos podem permanecer sem registro no banco; `media:cleanup` só processa ativos registrados como `Descartando`, portanto não recupera esses órfãos. A meta "0 objetos órfãos" não está garantida.
- **Situação:** parcial. Os testes citados cobrem os cenários simulados de proteção e compensação, mas não garantem ausência de órfãos quando a compensação falha, nem a recusa dos prefixos IPv6 citados.

## Categoria: Confiabilidade

### RNF09 — Integridade Transacional e Conformidade ACID

- **Fonte:** falha do banco (queda de conexão, violação de constraint) ou dado inválido enviado pelo cliente.
- **Estímulo:** falha no meio de uma escrita que envolve várias tabelas, por exemplo o cadastro de uma Obra (Obra, autores e papéis, gêneros, demografias, editoras originais, revistas e ativação da capa) ou a publicação de uma Edição (Edição e seus Volumes).
- **Artefato:** transações interativas do Prisma nos módulos `catalog`, `users` e `auth`; gatilhos de capa no PostgreSQL.
- **Ambiente:** produção, requisição de escrita administrativa ou de conta.
- **Resposta:** todas as escritas da operação ocorrem em uma única transação. Qualquer erro desfaz a transação inteira; a capa importada continua `Pendente` e pode ser reutilizada ou descartada. Alterações do catálogo que afetam publicação e capas também tomam um bloqueio consultivo global para evitar corridas entre requisições.
- **Medida:** 0 registros parciais ou órfãos após a falha.
- **Evidência:** o código usa `prisma.$transaction` nessas operações; testes unitários conferem que as escritas acontecem na mesma transação (`workAuthorPositions.unit.test.js`, `coverAssetLifecycle.unit.test.js`, `passwordRecovery.unit.test.js`, `adminModules.unit.test.js`) e que falhas de mídia acionam a remoção compensatória quando o armazenamento simulado permite apagar (`failureIsolationRegressions.unit.test.js`); a transação PostgreSQL não cobre o R2, e a compensação pode falhar (ver cenário de importação remota); `editionDerivedCover.test.js` confere a publicação da Edição e do Volume 1 na mesma transação contra o banco. Não há teste que injete falha no meio de uma gravação contra o banco real; o rollback em si é garantido pelo PostgreSQL.
- **Situação:** parcial. O uso de transação é verificado por teste; o rollback em falha real não tem teste dedicado.

### RNF10 — Integridade relacional rigorosa via constraints de banco

- **Fonte:** Administrador na interface ou falha de lógica no backend.
- **Estímulo:** exclusão de um registro pai com filhos vinculados, por exemplo uma Obra com Edições.
- **Artefato:** regras de exclusão do módulo `catalog` e chaves estrangeiras do PostgreSQL.
- **Ambiente:** operação normal, catálogo com registros relacionados.
- **Resposta:** a API recusa antes com 409 e mensagem de negócio (Obra pública ou com Edições, Edição pública ou com Volumes, Volume público). Se a lógica falhar, o banco atua como última defesa: a hierarquia Obra → Edição → Volume, as capas e os valores de opção usam `ON DELETE RESTRICT`; o tratador de erros converte a violação em 409 `DATA_INTEGRITY_CONFLICT`. Vínculos sem existência própria (sessões, perfis concedidos, tokens, vínculos da Obra, variantes de mídia) usam `CASCADE`.
- **Medida:** 0 exclusões que deixem registros órfãos; resposta 409 à tentativa.
- **Evidência:** o schema Prisma e as migrations declaram as ações referenciais; `prismaSchemaDrift.test.js` e `prismaSchemaReconciliation.unit.test.js` conferem que o banco segue o schema; `adminModules.unit.test.js` confere as recusas de exclusão.
- **Situação:** verificado por teste.

### RNF11 — Continuidade de dados e recuperação de desastres (DRP)

- **Fonte:** incidente no provedor do banco ou erro humano grave (script destrutivo em produção).
- **Estímulo:** perda ou corrupção do banco de produção.
- **Artefato:** banco PostgreSQL de produção no Neon e seus recursos de backup e restauração.
- **Ambiente:** produção com dados reais.
- **Resposta esperada:** a equipe restaura o banco a partir do histórico de recuperação do provedor ou de uma exportação lógica, reaplica as migrations se necessário e valida `/health/ready`.
- **Medida:** perda máxima de 24 horas de dados (RPO ≤ 24 h), retorno à operação em até 4 horas após a confirmação do incidente (RTO ≤ 4 h), retenção mínima de 7 dias.
- **Evidência:** o projeto não versiona rotina de backup, exportação lógica, procedimento de restauração nem registro de teste de restauração. Os recursos disponíveis dependem do plano contratado no Neon, que não está documentado no repositório.
- **Situação:** meta sem mecanismo de medição. Para cumprir o SERS, falta registrar as limitações do plano e, se necessário, criar exportações lógicas e testes periódicos de restauração.

### RNF12 — Observabilidade e Tratamento de Exceções (Graceful Failure)

- **Fonte:** requisição legítima que dispara uma falha inesperada.
- **Estímulo:** exceção não prevista durante o processamento, como perda de conexão com o banco ou tempo esgotado.
- **Artefato:** tratador global de erros (`errorHandler`), `notFoundHandler`, logger JSON estruturado e rotas de saúde.
- **Ambiente:** produção com tráfego normal.
- **Resposta:** o Express 5 encaminha erros síncronos e assíncronos ao tratador global. Respostas 5xx trazem `{ error: "Erro interno do servidor.", code }`, sem stack trace nem códigos internos do ORM. O log registra, em JSON, horário, identificador da requisição, método e rota registrada (sem query nem parâmetros), com stack trace apenas fora de produção; em rotas de autenticação o objeto de erro não vai ao log. Rotas inexistentes respondem 404 `ROUTE_NOT_FOUND`. `/health/ready` responde 503 quando o banco está indisponível.
- **Medida:** o processo continua ativo; 100% das falhas 5xx retornam JSON padronizado sem stack trace; 100% delas geram registro com rota, horário e mensagem.
- **Evidência:** `middlewares.unit.test.js` confere a resposta padronizada sem stack trace e a preservação de mensagem e código em erros 4xx encaminhados ao tratador; `failureIsolationRegressions.unit.test.js` confere que falhas de banco não vazam detalhes; `operations.unit.test.js` confere logs JSON sem stack trace em produção e o desligamento gracioso; `app.unit.test.js` confere 404 em JSON e o 503 de prontidão.
- **Limitações:** muitas respostas 4xx são produzidas diretamente pelos controladores e middlewares com `{ error }`, às vezes com `field`, sem `code` (por exemplo 401 de sessão, 403 de perfil e 401 de credenciais); o SERS exige `code` apenas nas falhas que passam pelo tratador global. Não há tratamento de `unhandledRejection` ou `uncaughtException` fora do ciclo de requisição.
- **Situação:** verificado por teste.

## Categoria: Usabilidade

### RNF13 — Feedback semântico e comunicação de estado (Toasts)

- **Fonte:** usuário enviando um formulário na Web.
- **Estímulo:** falha de conexão ou erro do servidor ao salvar, por exemplo, uma nova Obra.
- **Artefato:** `Toaster` global (Sonner) em `src/components/ui/sonner.tsx` e o tradutor de erros `getApiError`.
- **Ambiente:** produção, uso visual ou com leitor de tela.
- **Resposta:** a Web exibe um toast de erro com a mensagem segura enviada pela API (campo `error`) ou, na falta dela, uma mensagem em linguagem natural definida na tela. O Sonner anuncia as notificações em uma região `aria-live="polite"`. Todos os toasts, de erro ou de sucesso, ficam 5 segundos na tela e têm botão de fechar.
- **Medida:** toast de erro em vermelho, com mensagem em linguagem natural, anunciado por `aria-live`, visível por 5 segundos ou até ser fechado.
- **Evidência:** os testes das páginas conferem as mensagens enviadas ao toast (por exemplo `EditMangas.test.tsx`, `EditionForm.test.tsx`, `VolumeForm.test.tsx`, `ProtectedRoute.test.tsx`); `apiError.test.ts` confere o uso da mensagem da API e da alternativa. A duração, a região `aria-live` (fornecida pela biblioteca) e o contraste das cores não têm teste automatizado.
- **Situação:** implementado, sem teste da medida.

### RNF14 — Adaptabilidade de interface e navegação responsiva (Mobile-First)

- **Fonte:** usuário redimensionando a janela ou girando o dispositivo.
- **Estímulo:** largura da tela cruzando 768 px.
- **Artefato:** `PublicNav` e o layout principal (Tailwind CSS, estilos base para telas pequenas e prefixos `md:`/`lg:` para telas maiores).
- **Ambiente:** navegador em uso normal.
- **Resposta:** o CSS alterna a navegação sem recarregar a página nem chamar a API. Abaixo de 768 px aparece a barra inferior; de 768 px a 1023 px, uma barra lateral compacta com ícones; a partir de 1024 px, a barra lateral completa com rótulos.
- **Medida:** barra inferior visível em larguras menores que 768 px e barra lateral em larguras maiores ou iguais a 768 px.
- **Evidência:** as classes `md:hidden`, `hidden md:flex lg:hidden` e `hidden lg:flex` em `src/app/PublicNav.tsx`. `PublicNav.test.tsx` confere os itens de menu por tipo de sessão, mas o ambiente de teste não aplica media queries, então a troca por largura não é testada automaticamente.
- **Situação:** implementado, sem teste da medida (verificação manual).

### RNF15 — Padronização geométrica e resiliência visual de capas

- **Fonte:** origem de mídia devolvendo imagem fora do padrão ou indisponível.
- **Estímulo:** grade do catálogo com capas de proporção diferente de 2:3 e alguma capa respondendo erro (por exemplo 404).
- **Artefato:** componente `CatalogCover`, usado em `PublicWorkCard`, na pesquisa, nos detalhes de Obra, Edição e Volume e na seleção de Volumes.
- **Ambiente:** produção, grade densa de cards.
- **Resposta:** o contêiner tem proporção fixa 2:3 (`aspect-[2/3]`) e a imagem usa `object-cover`, cortando sem distorcer. Se a imagem falhar ao carregar ou não houver capa (por exemplo, Edição sem Volume 1 público), o componente mostra um placeholder no mesmo espaço ("Sem capa"; nas Edições da pesquisa e no detalhe da Edição o rótulo diverge, SERS 6.3), sem alterar o layout. As capas processadas já são geradas em 2:3 pela API.
- **Medida:** 100% das capas com proporção 2:3 e `object-fit: cover`; placeholder em 100% das falhas de carregamento; nenhum deslocamento de layout causado pelas capas.
- **Evidência:** `PublicWorkDetails.test.tsx`, `Pesquisa.test.tsx`, `PublicAuthorWorks.test.tsx` e `PublicVolumeDetails.test.tsx` conferem o placeholder após erro de carregamento ou ausência de capa. A ausência de deslocamento de layout (CLS) não é medida automaticamente.
- **Situação:** parcial. Placeholder verificado por teste; proporção garantida pelo código; CLS sem mecanismo de medição.

### RNF16 — Design semântico de Estados Vazios (Empty States)

- **Fonte:** usuário pesquisando ou filtrando o catálogo, ou Administrador consultando listas.
- **Estímulo:** combinação de filtros ou termo que não retorna registros.
- **Artefato:** componente `EmptyState` e as telas de listagem.
- **Ambiente:** operação normal.
- **Resposta:** a tela mostra uma mensagem diagnóstica ("Nenhuma Obra encontrada.", "Nenhuma Edição encontrada.") em vez de área em branco e, quando há ação útil, oferece o próximo passo, como "Limpar filtros" na pesquisa.
- **Medida:** 100% das visualizações sem registros mostram mensagem diagnóstica; ação de saída presente quando existir uma ação útil. Ícone ou ilustração é opcional.
- **Evidência:** `Pesquisa.test.tsx` confere a mensagem "Nenhuma Obra encontrada." e a ação "Limpar filtros"; testes de outras telas (detalhes públicos, Obras por Autor, gestão de contas e de opções) conferem mensagens de ausência de registros. Não há verificação sistemática de todas as telas.
- **Situação:** verificado por teste, nas telas principais.

## Resumo

| RNF | Situação |
| --- | --- |
| RNF01 | Meta com medição pendente (`load:test`) |
| RNF02 | Meta com medição pendente (`load:test`) |
| RNF03 | Parcial: 50 itens verificado; 2 MB com medição pendente |
| RNF04 | Parcial: sem binários no banco verificado; TTFB sem mecanismo |
| RNF05 | Implementado; custo 10 sem teste explícito |
| RNF06 | Verificado por teste, exceto o atributo Secure |
| RNF07 | Verificado por teste |
| RNF08 | Verificado por teste no login, com limitações de memória, janela fixa e IP atrás de proxy; cadastro e reenvio sem limite |
| Importação remota (RN0059) | Parcial: lacunas em prefixos IPv6 que embutem IPv4 e na limpeza quando a compensação falha |
| RNF09 | Implementado; rollback em falha real sem teste dedicado |
| RNF10 | Verificado por teste |
| RNF11 | Meta sem mecanismo de medição |
| RNF12 | Verificado por teste |
| RNF13 | Implementado, sem teste da medida |
| RNF14 | Implementado, sem teste da medida |
| RNF15 | Parcial: placeholder verificado; CLS sem mecanismo |
| RNF16 | Verificado por teste nas telas principais |
