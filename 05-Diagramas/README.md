# Diagramas do CoMangá

Esta pasta reúne os diagramas visuais do CoMangá e a fonte textual que orienta sua atualização.

A referência vigente é o [modelo textual dos diagramas](./Modelo-Textual-para-IA.md). Ele descreve a arquitetura, o domínio, os fluxos e as rotas do sistema atual. Os PNGs abaixo foram produzidos antes das mudanças de perfis, catálogo e mídia e estão **desatualizados**: não devem ser usados como descrição do sistema até serem regenerados a partir do modelo textual.

## Inventário

| Diagrama | Arquivo | Finalidade | Fonte editável | Estado |
| --- | --- | --- | --- | --- |
| Sequência - Login Stateful | [PNG](./1%20-%20Diagrama%20de%20Sequ%C3%AAncia%20-%20Login%20Stateful%20-%20CoMang%C3%A1.png) | Mostrar o login com sessão persistida no servidor e cookie HttpOnly. | Não versionada (o traço indica PlantUML). | Desatualizado |
| Casos de Uso | [PNG](./2%20-%20Diagrama%20de%20Casos%20de%20Uso%20-%20CoMang%C3%A1.png) | Mostrar atores e funcionalidades do produto. | Não versionada. | Desatualizado |
| DER Físico | [PNG](./3%20-%20DER%20F%C3%ADsico%20-%20CoMang%C3%A1.png) | Mostrar tabelas, colunas e chaves do PostgreSQL. | Não versionada (gerado no dbdiagram.io). | Desatualizado |
| Classes ORM Prisma | [PNG](./4%20-%20Diagrama%20de%20Classes%20ORM%20Prisma%20-%20CoMang%C3%A1.png) | Mostrar os modelos Prisma e suas relações. | Não versionada (o traço indica PlantUML). | Desatualizado |

Nenhum arquivo-fonte (PlantUML, DBML, Draw.io ou similar) está no repositório. Ao regenerar um diagrama, recomenda-se versionar a fonte ao lado do PNG. Os blocos Mermaid do modelo textual podem servir de ponto de partida.

## O que cada diagrama precisa mudar

### 1 - Diagrama de Sequência - Login Stateful

- Acrescentar, na API e antes do controlador de login, o limitador de tentativas em memória: após 5 falhas do mesmo IP em 5 minutos, a resposta é 429 sem consulta ao banco.
- Incluir a conta Bloqueada no ramo de resposta 403, junto com a conta Pendente.
- No ramo de sucesso, mostrar a transação com bloqueio da linha do usuário, a resolução do perfil ativo inicial (perfil preferido, se ainda concedido; senão Usuário Padrão) e a gravação de `active_profile_id` na sessão.
- Mostrar os atributos do cookie: HttpOnly, SameSite=Strict por padrão e Secure em produção.
- Mostrar que a resposta traz os perfis concedidos e o perfil ativo, e que o cliente atualiza o `AuthContext` com eles.
- Trocar o participante "AuthController", que não existe, pelo módulo `auth` da API. Manter "PostgreSQL (Neon)".
- Opcional: acrescentar a validação da sessão em uma rota protegida (hash do cookie, sessão não revogada, conta Ativada, perfil ativo ainda concedido).

### 2 - Diagrama de Casos de Uso

- Substituir o ator único "Colecionador" por Visitante e Conta autenticada, e mostrar o Administrador como perfil ativo de uma conta, não como pessoa separada.
- Trocar "Gerenciar Nível de Acesso (Role)" por "Conceder/remover perfil Administrador de outra conta".
- Acrescentar: trocar perfil ativo, reenviar ativação, redefinir senha, alterar senha, excluir conta, consultar Obras por Autor e consultar detalhes de Obra, Edição e Volume com navegação entre Volumes.
- Detalhar "Gerenciar Catálogo Geral" em Obras, Edições, Volumes, "Gerenciar opções" (sem tipos de obra e gêneros, que são valores fixos do sistema) e importar capa por URL.
- Marcar "Gerenciar Estante Digital", "Gerenciar Lista de Desejos" e "Visualizar Checklist" como planejados, sem persistência.
- Registrar que, com o perfil Administrador ativo, a conta não usa as páginas públicas do catálogo.

### 3 - DER Físico

- Acrescentar em `users`: `birth_date` e `preferred_profile_id`.
- Acrescentar em `sessions`: `active_profile_id`.
- Acrescentar as tabelas `profiles`, `user_profiles` e `password_reset_tokens`.
- Acrescentar as tabelas do catálogo: `works`, `editions` (com `cover_type_id` e `format_id` anuláveis, sem `paper_id`, tipo de edição ou capa própria), `edition_papers` (chave composta Edição/miolo e `position`) e `volumes`.
- Acrescentar as tabelas de vínculo da Obra: `work_authors` (com `position`), `work_author_roles`, `work_genres`, `work_demographies`, `work_original_publishers` e `work_serialization_magazines` (ambas com `position`).
- Acrescentar `domain_option_categories`, `domain_option_values` (com `code`, `system_managed`, `adult_only`, `position`, `active`) e `domain_option_value_dependencies`.
- Acrescentar `media_assets` e `media_variants`, com `works.cover_asset_id` e `volumes.cover_asset_id` únicos e obrigatórios.
- Não desenhar `edition_type_id`, `original_volume_count` nem `editions.cover_asset_id`.

### 4 - Diagrama de Classes ORM Prisma

- Acrescentar os modelos `Profile`, `UserProfile`, `PasswordResetToken`, `Work`, `Edition`, `EditionPaper`, `Volume`, `WorkAuthor`, `WorkAuthorRole`, `WorkGenre`, `WorkDemography`, `WorkOriginalPublisher`, `WorkSerializationMagazine`, `DomainOptionCategory`, `DomainOptionValue`, `DomainOptionValueDependency`, `MediaAsset` e `MediaVariant`.
- Acrescentar em `User`: `birthDate`, `preferredProfileId` e as relações `userProfiles`, `passwordResetTokens` e `mediaAssets`. Acrescentar em `Session`: `activeProfileId`.
- Mostrar a hierarquia Obra → Edição → Volume e as capas obrigatórias de `Work` e `Volume`; `Edition` não tem capa.
- Distinguir o campo legado `nivelAcesso` dos perfis concedidos.

## Diagramas que ainda não existem

O modelo textual já descreve a implantação (Vercel, Render, Neon, Cloudflare R2 e Resend) e os módulos internos da API e da Web. Se a equipe quiser diagramas de implantação ou de componentes, eles devem ser criados a partir das seções "Arquitetura de implantação", "Estrutura interna da API" e "Estrutura interna da Web".
