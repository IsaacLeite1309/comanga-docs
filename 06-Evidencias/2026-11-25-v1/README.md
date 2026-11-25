# Evidências da revalidação — 25/11/2026

- [Validação parcial no site público](validacao-publica.md): ações e resultados observados sem autenticação.
- [Confirmação dos testes manuais](confirmacao-testes-manuais.md): ações confirmadas pelo responsável em Windows e Google Chrome, em 21/11/2026; ressalva para os passos alterados posteriormente.
- [API: testes unitários e cobertura](api-unitarios.log): 57 suítes e 792 testes aprovados.
- [API: lint, arquitetura e build](comanga-api-verificacoes.log): os três comandos terminaram com código 0.
- [Frontend: testes e cobertura](frontend-testes.log): 69 arquivos e 628 testes aprovados.
- [Frontend: lint, arquitetura e build](comanga-web-verificacoes.log): os três comandos terminaram com código 0.
- [Frontend: log do GitHub Actions](frontend-ci.log) e [execução no GitHub](https://github.com/IsaacLeite1309/comanga-web/actions/runs/45690820696): workflow de qualidade concluído com sucesso em 25/11/2026.
- [API: execução completa no Neon de teste](api-integracao-neon.log): 94 de 95 suítes e 1.299 de 1.300 testes aprovados; uma falha por limite de 30 segundos no teste de migração. No [reteste isolado](api-neon-reteste-migracao.log), 4 de 4 testes passaram em 45,5 s com limite de 180 segundos. O arquivo [jest-neon-reteste.js.txt](jest-neon-reteste.js.txt) registra a configuração usada no reteste; a execução completa não foi repetida com esse limite.
- [API: suíte completa no GitHub Actions](api-integracao-ci.log): 95 suítes e 1.300 testes aprovados em PostgreSQL 16 isolado. [Execução no GitHub](https://github.com/IsaacLeite1309/comanga-api/actions/runs/45690820953).

O código testado localmente corresponde aos commits `23b59c48115031b8917bc54f0ee6c1c1b24f981d` da API e `81c3e38eaf5e118f0f7bb6f3fb47f2a59290203b` do frontend. Os READMEs possuíam alterações locais; elas não afetam o código das aplicações. Os testes automatizados não substituem os passos manuais não executados.

O [manifesto](manifesto.json) registra os commits e os hashes SHA-256 dos registros concluídos.

Na suíte completa da API, o log do CI registra, entre outros, os seguintes arquivos aprovados:

- Cadastro, ativação, login, logout e recuperação: `authRegister.test.js`, `authActivate.test.js`, `authLoginSession.test.js`, `authLogout.test.js` e `authRecovery.test.js`.
- Perfis e conta: `accessProfiles.test.js`, `userProfile.test.js`, `userAccountSettings.test.js` e `userDeleteAccount.test.js`.
- Administração e catálogo: `adminUsers.test.js`, `adminOptions.test.js`, `adminWorkClassifications.test.js`, `editionDerivedCover.test.js` e `catalogIntegrityRegressions.test.js`.
- Catálogo público e lançamentos: `publicCatalog.test.js`, `publicCalendar.test.js` e `adultContentAccessBoundaries.test.js`.
- Coleção e lista de desejos: `personalCatalog.test.js` e os testes unitários `personalMutations.unit.test.js` e `personalReaders.unit.test.js`.

Os nomes completos e o resultado de todos os arquivos constam em [api-integracao-ci.log](api-integracao-ci.log). A aprovação de uma suíte automatizada não significa que cada passo do roteiro manual foi executado.
