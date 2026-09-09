# Cenários ATAM do CoMangá

Cenários ATAM para os RNFs do CoMangá:

## Categoria: Desempenho

### RNF01: Desempenho de tempo de resposta do motor de busca de Obras e Edições

Estímulo: Execução intensiva de requisições de buscas paginadas e complexas com termos textuais variados, simulando um pico de leitura simultânea ao catálogo de Obras e Edições.

Fonte: Múltiplos usuários externos simultâneos operando via chamadas HTTP à API do motor de busca.

Ambiente: Sistema sob carga normal de operação operacional, executado em um ambiente de rede cliente-servidor estável, com latência de tráfego de rede comprovadamente garantida em até 100ms.

Resposta: A arquitetura do Back-end em conjunto com o banco de dados processa as consultas, utilizando estratégias de indexação para evitar varreduras completas nas tabelas (full table scans), serializa a paginação e devolve os dados de forma assíncrona sem travar a fila de processamento principal.

Medida: O processamento e retorno dos resultados da busca devem ser concluídos em um tempo máximo de 500 milissegundos (ms) para o percentil 95 (P95) de todas as requisições realizadas no período aferido.

### RNF02 - Capacidade de vazão e concorrência da API Pública

Estímulo: Recebimento abrupto de um pico de tráfego de leitura, atingindo a marca de 20 requisições simultâneas por segundo nas rotas de consulta pública da Vitrine.

Fonte: Múltiplos usuários externos (ex: pico de acessos simultâneos gerado por estudantes durante a apresentação acadêmica do projeto ou após divulgação em massa do link do catálogo).

Ambiente: Sistema operando em estado normal de produção, com o banco de dados populado e sem rotinas de manutenção ou backup concorrentes.

Resposta: A arquitetura do Back-end gerencia a concorrência eficientemente, enfileirando e resolvendo as conexões com o banco de dados sem esgotar o pool de conexões ou causar o encerramento abrupto do processo da API.

Medida: Sustentação do teto de carga de 20 RPS garantindo que a taxa de falhas de servidor (respostas com código HTTP 5xx) permaneça estritamente inferior a 1%.

### RNF03 - Eficiência de tráfego e limite de paginação

Estímulo: Requisição de listagem da coleção de Obras buscando retornar o catálogo inteiro ou com parâmetro de paginação explícito solicitando mais de 50 itens simultâneos.

Fonte: Cliente externo (Frontend Web) ou script de consulta disparado diretamente contra a API do sistema.

Ambiente: Sistema em operação normal, conectado a um banco de dados de produção contendo um grande volume de registros (centenas ou milhares de mangás e edições cadastrados).

Resposta: O Back-end intercepta a requisição, ignora o excesso solicitado, aplica a restrição de paginação compulsória no nível da consulta ao banco de dados e serializa a resposta truncada.

Medida: O payload da resposta HTTP (JSON) gerado não ultrapassa o limite de 2 MB de tamanho absoluto e o array de resultados contém, no máximo, 50 itens por página.

### RNF04 - Desempenho de entrega de mídia estática (Capas)

Estímulo: Carregamento da página do catálogo (Vitrine), disparando requisições paralelas para a exibição de múltiplas capas de mangás simultaneamente na interface gráfica.

Fonte: Navegadores web de usuários finais acessando a listagem pública de obras.

Ambiente: Operação normal do sistema em ambiente de produção, durante a navegação fluida em um grid contendo dezenas de referências visuais de obras.

Resposta: O Back-end retorna exclusivamente URLs derivadas de chaves internas de mídia associadas ao catálogo. A carga de requisição dos arquivos de mídia é totalmente roteada para o serviço de Object Storage ou CDN configurado pelo CoMangá, isolando o banco de dados relacional do processamento de binários.

Medida: O Tempo até o Primeiro Byte (TTFB) da entrega das capas de mangá não excede 300 milissegundos no percentil 90 (P90) das requisições, comprovando-se a

ausência de armazenamento de BLOBs ou conversões base64 no banco de dados relacional.

## Categoria: Segurança

### RNF05 - Armazenamento irreversível de credenciais (Hashing de Senhas)

Estímulo: Acesso não autorizado de leitura direta ou auditoria de segurança rigorosa nas tabelas de usuários do banco de dados relacional visando a extração de credenciais.

Fonte: Ator malicioso (atacante) explorando uma vulnerabilidade de infraestrutura ou auditor de segurança com acesso administrativo.

Ambiente: Sistema em produção (ou ambiente análogo), contendo uma base de dados populada com múltiplos registros ativos de usuários autenticáveis.

Resposta: O banco de dados expõe estritamente as strings de hash correspondentes às senhas. A arquitetura inviabiliza a engenharia reversa ou ataques de rainbow tables, protegendo o texto plano original graças ao custo computacional e ao salting dinâmico aplicado previamente pela camada de aplicação.

Medida: 100% das senhas armazenadas e inspecionadas utilizam exclusivamente o algoritmo Bcrypt com um fator de trabalho (Salt Rounds) mínimo de 10,

comprovando-se a existência de 0 (zero) registros em texto plano, criptografia reversível, MD5 ou SHA-1.

### RNF06 - Gerenciamento de sessão stateful e transporte seguro

Estímulo: Tentativa de extração do identificador de sessão (Session ID) via execução de script JavaScript malicioso no navegador do cliente (ataque XSS), combinada com uma ação de revogação imediata e compulsória do acesso deste usuário por um administrador.

Fonte: Ator malicioso (script injetado na interface do cliente) e Administrador do Sistema (comando de revogação/Logout no servidor).

Ambiente: Sistema em operação de produção, com o usuário-alvo legitimamente autenticado, ativo na plataforma e trafegando sob conexão criptografada (HTTPS).

Resposta: O navegador do cliente impede o acesso do script ao cookie de sessão. Simultaneamente, a arquitetura Back-end (Stateful) destrói o estado de sessão na memória/banco de dados, invalidando o Session ID e rejeitando instantaneamente qualquer nova requisição de acesso, mesmo que o invasor possuísse o cookie.

Medida: Comprovação da presença integral das diretivas HttpOnly, Secure e

SameSite=Strict no cookie de sessão, resultando em 0% de exposição do

Session ID a scripts de terceiros, e garantia de bloqueio de 100% das requisições subsequentes à ação de revogação no servidor.

### RNF07 - Controle de Acesso Baseado em Funções (RBAC) via Middleware

Estímulo: Tentativa de execução de uma operação restrita de mutação de dados (ex: exclusão de uma Obra do catálogo ou alteração de privilégios) em uma rota administrativa da API.

Fonte: Usuário autenticado na plataforma, porém portador de uma role (função) com privilégios básicos/padrão, sem direitos administrativos.

Ambiente: Sistema em operação normal de produção, com o usuário mantendo uma sessão stateful ativa e válida no servidor.

Resposta: O Middleware da API intercepta a requisição, recupera a role da sessão e identifica a insuficiência de privilégios. A arquitetura aplica o padrão Fail-Fast, abortando o fluxo imediatamente antes de acionar qualquer controlador, regra de negócio ou query de banco de dados.

Medida: Retorno instantâneo do código de erro HTTP 403 (Forbidden) ao cliente, com a garantia técnica de 0 (zero) execuções de regras de negócio ou transações no banco de dados decorrentes da requisição não autorizada.

### RNF08 - Limitação de Taxa de Autenticação (Rate Limiting)

Estímulo: Disparo automatizado de requisições sucessivas de login com credenciais incorretas, excedendo o limite de 5 tentativas dentro de uma janela temporal de 5 minutos.

Fonte: Script malicioso (ataque de força bruta ou credential stuffing) originado de um único endereço IP externo.

Ambiente: Sistema operando em ambiente de produção público, com rotas de autenticação expostas à internet e conectadas ao banco de dados principal.

Resposta: O Middleware de segurança intercepta as requisições, identifica a violação do limite estipulado para aquele IP específico e aborta preemptivamente as novas tentativas de acesso, protegendo o servidor e o banco de dados contra carga inútil e vazamento de senhas.

Medida: Bloqueio imediato a partir da 6ª tentativa, com retorno obrigatório do código de erro HTTP 429 (Too Many Requests) para o IP agressor e execução de 0 (zero) consultas ao banco de dados até a expiração natural da janela deslizante de 5 minutos.

## Categoria: Confiabilidade

### RNF09 - Integridade Transacional e Conformidade ACID

Estímulo: Ocorrência de uma falha crítica (ex: queda de conexão com o banco ou violação de restrição de chave) durante a gravação das "Edições" ou "Autores", imediatamente após o sistema já ter inserido com sucesso a tabela principal de "Obra".

Fonte: Instabilidade na infraestrutura do SGBD ou erro de validação de dados enviado pelo cliente durante a operação.

Ambiente: Sistema em ambiente de produção recebendo uma requisição de cadastro multi-entidade (payload complexo) pela API.

Resposta: A camada de persistência detecta a quebra da unidade lógica, aborta a transação em andamento e aciona um Rollback automático, desfazendo a inserção prévia da "Obra" antes de retornar o erro ao cliente.

Medida: Execução de Rollback total garantindo a persistência de 0 (zero) registros órfãos ou parciais no banco de dados referentes à transação falha.

### RNF10 - Integridade relacional rigorosa via constraints de banco

Estímulo: Tentativa de execução de um comando de exclusão física (DELETE) em um registro "pai" (ex: uma Obra) que ainda possui múltiplos registros "filhos" (ex: Edições ou volumes) vinculados a ele por chave estrangeira.

Fonte: Ação de exclusão disparada por um administrador na interface gráfica ou uma falha na lógica de orquestração do Back-end (bug) que tentou remover a entidade principal sem limpar suas dependências previamente.

Ambiente: Sistema em operação normal de produção, com o SGBD contendo um catálogo histórico vasto e entidades fortemente acopladas através de relacionamentos relacionais.

Resposta: A camada física do banco de dados atua como última linha de defesa, aciona a diretiva ON DELETE RESTRICT, bloqueia sumariamente o comando de exclusão na origem e aborta a transação, impedindo o efeito dominó.

Medida: Interrupção imediata da operação com retorno obrigatório de erro de integridade referencial nativo do SGBD, garantindo 0 (zero) perdas acidentais de dados históricos ou criação de registros órfãos.

### RNF11 - Continuidade de dados e recuperação de desastres (DRP)

Estímulo: Ocorrência de uma falha catastrófica resultando na corrupção total ou perda física irreversível da base de dados principal do catálogo.

Fonte: Incidente crítico na infraestrutura do provedor de nuvem (SGBD) ou erro humano de alta severidade (ex: execução de um script destrutivo sem validação no banco de produção).

Ambiente: Sistema operando em ambiente de produção contendo um histórico vital de dados estruturados, com a rotina de snapshots automatizados ativada no provedor de hospedagem.

Resposta: A equipe ou o administrador do sistema aciona o Plano de Recuperação de Desastres (DRP), acessando os cofres de backup do provedor em nuvem e iniciando a restauração (rollback) a partir de um snapshot íntegro contido no histórico de retenção de 7 dias.

Medida: O sistema é reestabelecido para o último estado válido perdendo no máximo 24 horas de inserção de dados (RPO ≤ 24 horas), com o serviço retornando à operação completa em um limite de até 4 horas após a detecção do incidente (RTO ≤ 4 horas).

### RNF12 - Observabilidade e Tratamento de Exceções (Graceful Failure)

Estímulo: Ocorrência de uma exceção interna imprevista (ex: falha de processamento de dados, timeout ou perda de conectividade repentina com o banco de dados) durante o ciclo de vida de uma requisição HTTP.

Fonte: Requisição legítima de um cliente (usuário externo ou interface Front-end) que desencadeia a falha lógica ou de infraestrutura na camada do servidor.

Ambiente: Sistema rodando em ambiente de produção público, recebendo tráfego normal, onde o encapsulamento da infraestrutura e a estabilidade do processo são críticos.

Resposta: O Global Error Handler (Middleware de exceções) intercepta a falha, impedindo o crash e o encerramento do processo do servidor. A arquitetura isola o erro técnico gravando-o no serviço de observabilidade interno e devolve uma resposta higienizada para o solicitante.

Medida: O servidor mantém a operação contínua, garantindo que 100% das falhas retornem um objeto JSON padronizado com código de erro amigável, provando 0%

de vazamento de stack trace para o cliente e 100% de registro técnico (rota, horário, mensagem) nos logs do sistema para diagnóstico.

## Categoria: Usabilidade

### RNF13 - Feedback semântico e comunicação de estado (Toasts)

Estímulo: Ocorrência de uma falha de conexão ou erro interno de servidor durante a tentativa de submissão de um formulário de cadastro (ex: salvamento de uma nova Obra no catálogo).

Fonte: Ação de submissão disparada pelo usuário na interface gráfica (Front-end), resultando em uma resposta técnica de erro (ex: HTTP 500) devolvida pela API.

Ambiente: Sistema em produção, com o usuário interagindo com a interface Web visualmente ou através de tecnologias assistivas (leitores de tela).

Resposta: A arquitetura do Front-end intercepta o erro técnico gerado pelo Back-end, omite jargões de banco de dados ou do compilador, e renderiza dinamicamente um componente de notificação flutuante na interface para comunicar a falha de forma compreensível.

Medida: Renderização de um Toast de erro (cor Vermelha com contraste validado pela WCAG) contendo mensagem exclusivamente em linguagem natural (ex: "Não foi possível conectar ao servidor"), estruturado com o atributo HTML aria-live, mantendo-se visível na interface por exatos 5 segundos ou até ser fechado manualmente pelo usuário.

### RNF14 - Adaptabilidade de interface e navegação responsiva (Mobile-First)

Estímulo: Redimensionamento dinâmico da janela do navegador (ou rotação de tela em um dispositivo móvel/tablet) que force a viewport a cruzar o limiar de largura estabelecido.

Fonte: Ação do usuário final manipulando o navegador da Web ou alterando a orientação física do seu dispositivo de acesso.

Ambiente: Sistema renderizando a interface gráfica no Front-end (navegador do cliente) em operação normal de uso, com os componentes de navegação estrutural ativos.

Resposta: O motor de renderização do navegador reavalia a folha de estilos baseada em Media Queries, transicionando instantaneamente os componentes do layout sem recarregar a página ou realizar requisições ao servidor, priorizando o processamento do CSS móvel nativo.

Medida: Transição exata e comprovável da arquitetura de navegação no ponto de quebra de 768px: exibição obrigatória da Bottom Navigation Bar em larguras < 768px, e transição imediata para a Sidebar em larguras ≥ 768px.

### RNF15 - Padronização geométrica e resiliência visual de capas

Estímulo: Renderização simultânea de obras no catálogo cujas capas internas possuam proporções geométricas anômalas (ex: 16:9 panorâmico ou 1:1 quadrado) e cuja entrega temporariamente retorne erro HTTP 404.

Fonte: Serviço de mídia configurado pelo CoMangá entregando arquivos fora do padrão físico ou apresentando indisponibilidade temporária de um recurso interno associado.

Ambiente: Sistema em operação de produção, com o usuário final navegando na Vitrine/Catálogo através de um dispositivo com a interface gráfica exibindo um grid denso de componentes MangaCard.

Resposta: O motor CSS do Front-end isola as dimensões originais das imagens, forçando o corte sem distorção. Paralelamente, o componente intercepta a indisponibilidade temporária de um recurso interno no lado do cliente e aciona o estado de resiliência visual, injetando o placeholder estático antes que o layout sofra qualquer variação abrupta.

Medida: Manutenção absoluta do layout (ausência de Cumulative Layout Shift decorrente das capas), atestando que 100% dos componentes MangaCard mantiveram a restrição aspect-ratio: 2/3 com object-fit: cover (sem stretch) e exibiram o placeholder padronizado para todas as requisições de recursos internos já associados que retornaram erro 404 temporário.

### RNF16 - Design semântico de Estados Vazios (Empty States)

Estímulo: Aplicação de múltiplos filtros restritivos simultâneos ou inserção de um termo de busca estrito e inexistente, resultando em um conjunto de dados nulo (array vazio) devolvido pela camada de dados.

Fonte: Ação de consulta explícita disparada pelo usuário final interagindo com o motor de busca ou filtros da interface gráfica (Front-end).

Ambiente: Sistema operando em ambiente de produção, com o usuário aguardando o carregamento de uma visualização baseada em coleções (ex: Estante Pessoal ou Catálogo Geral).

Resposta: O Front-end intercepta o payload vazio ([]) enviado pela API e previne a renderização de uma tabela vazia ou tela em branco, montando dinamicamente o componente de Estado Vazio (<EmptyState />) no contêiner da página.

Medida: Comprovação de que 100% das instâncias de ausência de dados renderizam o componente contendo a tríade estrutural obrigatória: (1) ilustração/ícone de feedback visual, (2) texto de diagnóstico contextualizado em linguagem natural, e (3) elemento interativo de Rota de Fuga (Call to Action, ex:

"Limpar Filtros").
