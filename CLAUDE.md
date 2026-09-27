# Repositório APEXlang — Customers

Monorepo com dois artefatos irmãos que evoluem juntos:

- [customers/](customers/CLAUDE.md) — aplicação Oracle APEX `CUSTOMERS`, gerada/mantida com o
  skill `apex/apexlang` (Claude Code). Ver `customers/CLAUDE.md` para as regras
  completas de geração de APEXlang.
- [Database/](Database/) — scripts DDL exportados pelo projeto **Database** do
  Oracle JDeveloper, registrando a evolução do schema alvo da aplicação.

Este arquivo raiz é o **project root** para o skill `apexlang`: comandos
`node tools/apexctl.mjs ...` devem ser executados com o diretório de trabalho
aqui (`D:\Repos\APEXlang`), nunca dentro da pasta do skill — ver detalhes em
`customers/CLAUDE.md`. Artefatos de planejamento gerados pelo skill
(`.apexlang/application-spec.md`, `.apexlang/app-ux-contract.json`) também
nascem aqui na raiz, não dentro de `customers/`.

## Convenção do diretório `Database/`

- Arquivos `DDL_v001.sql`, `DDL_v002.sql`, ... gerados pelo JDeveloper a cada
  evolução do modelo.
- **São diffs incrementais**, não snapshots completos: cada arquivo contém só
  os `CREATE`/`ALTER`/`DROP` novos daquela versão. A verdade atual do schema é
  a aplicação, em ordem numérica, de **todos** os arquivos — nunca assuma que
  o arquivo de maior número sozinho descreve o schema inteiro.
- Ao usar o schema como contexto para gerar APEXlang, leia todos os
  `DDL_vNNN.sql` em ordem ascendente e componha o estado atual (tabelas,
  colunas, PK/FK, constraints) antes de tratar como autoritativo.
- Nunca editar um `DDL_vNNN.sql` já commitado — ele é histórico. Uma correção
  de schema vira um novo `DDL_vNNN+1.sql` gerado pelo JDeveloper.

## Convenções observadas no schema (válidas até serem contrariadas por uma
## versão mais nova do DDL)

- PK surrogate `RAW(16)`, preenchida por trigger `BEFORE INSERT` via
  `SYS_GUID()` — nunca modelar como item obrigatório em formulário de
  criação.
- Colunas de auditoria `CREATED_AT`, `CREATED_BY`, `UPDATED_AT`, `UPDATED_BY`
  mantidas por trigger via `APEX_UTIL.GET_SESSION_STATE('APP_USER')` — sempre
  display-only/hidden em formulários, nunca editáveis pelo usuário.
- Prefixo de tabela `TST_` em uso; confirmar com o usuário se é definitivo
  antes de reutilizar o prefixo em objetos novos.

Regras completas de geração/validação de APEXlang, SQLcl e o app `CUSTOMERS`
estão em [customers/CLAUDE.md](customers/CLAUDE.md).
