# APEXlang — Customers (projeto demo)

🌐 Também disponível em: [English](README.en.md) | [Español](README.es.md)

## O que é este projeto

Um projeto demo/teste para explorar o desenvolvimento de aplicações Oracle
APEX usando **APEXlang** (uma DSL textual para APEX) junto com o **Claude
Code** como agente de IA, fechando o ciclo com **SQLcl** e a **extensão
Oracle SQL Developer para VS Code** (que importa/publica o app direto no
workspace APEX a partir do editor).

Não é um projeto de produção — é um laboratório para validar o fluxo de
ponta a ponta: descrever uma funcionalidade em linguagem natural → o agente
gera `.apx` a partir de contratos de gramática e do schema real do banco →
importa no APEX pelo VS Code → testa no navegador → corrige o que quebrar →
registra o aprendizado para não repetir o mesmo erro na próxima página.

## O que já foi feito até aqui

- App Oracle APEX `CUSTOMERS` criado e publicado (workspace `ZONA`, app id
  `100`).
- Schema inicial (`Database/DDL_v001.sql`): 5 tabelas — clientes, tipos de
  telefone, representantes de venda, telefones do cliente e pedidos — com
  chave primária `RAW(16)`/`SYS_GUID()`, colunas de auditoria mantidas por
  trigger, chaves estrangeiras e constraints de unicidade.
- Primeira funcionalidade pedida via prompt e implementada de ponta a
  ponta: manutenção de **Tipos de Telefone** — página com Interactive Grid
  (edição só de Nome/Descrição, Código é só-leitura, sem insert/delete
  inline), criação de registro via página modal separada, entrada de menu
  e de breadcrumb.
- Convenções do projeto e vários problemas reais de runtime encontrados no
  caminho (e suas causas) documentados nos `CLAUDE.md`, para que toda
  página nova já nasça sem repetir os mesmos erros.

## Como o repositório está organizado

```
APEXlang/
├── CLAUDE.md                        # convenções gerais do repo p/ o agente de IA
├── README.md / README.en.md / README.es.md
├── customers/                       # app Oracle APEX (raiz do app — ver abaixo)
├── Database/                        # evolução do schema (DDL) + dicionário de dados
└── Prompts/                         # histórico dos pedidos de funcionalidade, por app
```

### `customers/` — app Oracle APEX

Contém os artefatos `.apex`/`.apx` que o skill APEXlang gera e que a
extensão Oracle SQL Developer importa/exporta direto no workspace APEX.
Diferente do padrão mais comum do skill (`applications/<app>/`), aqui
`customers/` já **é** a raiz do app.

- `application.apx` — definição da aplicação (nome, tema, autenticação,
  navegação).
- `pages/` — um arquivo `.apx` por página (`p00001-home.apx`,
  `p00002-phone-types.apx`, `p00003-phone-types-create.apx`, ...).
- `shared-components/` — breadcrumbs, listas de menu, autenticação e
  autorização, tema, LOVs, arquivos estáticos.
- `supporting-objects/` — scripts de instalação/desinstalação do app.
- `deployments/default.json` — workspace e app id de destino (`ZONA` /
  `100`).
- `apex-exports/` — backups automáticos em `.sql` que a extensão gera antes
  de cada Import; é saída de segurança, não fonte de verdade.

### `Database/` — schema e dicionário de dados

É a fonte de verdade do modelo de dados, alimentada pelo projeto Database
do Oracle JDeveloper.

- `DDL_vNNN.sql` — um arquivo por versão, gerado pelo JDeveloper. **Cada
  arquivo é um diff incremental** (só o que mudou naquela versão), não um
  snapshot completo do schema — a verdade atual é a soma de todos os
  arquivos aplicados em ordem.
- `Tables/<TABELA>.md` — um dicionário de dados por tabela: nome do campo
  (igual ao nome da coluna no banco), label de exibição em português e uma
  descrição. Complementa a DDL com os fatos que ela não carrega (label,
  descrição, indicação de campo oculto/não editável) e é tratado como
  metadado autoritativo, no mesmo nível da DDL — não como um simples
  comentário solto.

### `Prompts/` — histórico de pedidos

Um arquivo por funcionalidade pedida, nunca sobrescrito:
`Prompts/<app>/NNN_NomeDaFuncionalidade.md`. Cada arquivo registra o pedido
original e, quando aplicável, os problemas encontrados durante a
implementação e como foram corrigidos — funciona como um changelog vivo por
funcionalidade.

## Ferramentas envolvidas

- **Claude Code** com o skill `apexlang`, que gera e valida `.apx` a partir
  de contratos de gramática e de metadados reais do compilador Oracle APEX
  (não "inventa" sintaxe).
- **SQLcl** (`D:\Bin\sqlcl`) — usado pelo skill para consultar metadados do
  compilador e pela extensão do VS Code para conectar ao banco.
- **Extensão Oracle SQL Developer para VS Code** — reconhece arquivos
  `.apx`/`.apex` e publica (Import) o app no workspace `ZONA` direto do
  editor.

## Convenções e detalhes para quem for mexer no projeto

As regras de como gerar código, como nomear as coisas (inglês para escopo
técnico/identificadores, português para o que o usuário final vê) e a
lista de armadilhas de runtime já descobertas ao longo do projeto estão
documentadas em:

- [`CLAUDE.md`](CLAUDE.md) — convenções gerais do repositório.
- [`customers/CLAUDE.md`](customers/CLAUDE.md) — convenções específicas do
  app `CUSTOMERS`.
