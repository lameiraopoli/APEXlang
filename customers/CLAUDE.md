# Projeto: Customers (Oracle APEX)

## Visão geral
Aplicação Oracle APEX `CUSTOMERS` (name: "Customers"), gerada e mantida com o
skill `apex/apexlang` (Claude Code). Tema Universal Theme, autenticação
`oracle-apex-accounts`.

- Workspace APEX: `ZONA` (fonte: [deployments/default.json](deployments/default.json))
- Schema principal / owner do banco: `ZONA` (confirmado pelo usuário).
- App id numérico: `100` (fonte: [deployments/default.json](deployments/default.json),
  atribuído no primeiro import no workspace `ZONA`).

## Estrutura real deste projeto (IMPORTANTE — foge do padrão do skill)
O skill `apexlang` assume por padrão apps em `applications/<app>/`. **Este
repositório não usa esse padrão**: `customers/` já É a raiz do app,
diretamente na raiz do projeto. O probe do skill classifica isso como
"nonstandard app candidate" e pediria confirmação — **considere isso já
confirmado**: o app-path a usar em qualquer comando `--app-path` é
`customers` (relativo à raiz do repo) ou o caminho absoluto
`D:\Repos\APEXlang\customers`. Nunca criar/assumir uma pasta `applications/`.

Dentro de `customers/`:
- `.apex/`, `application.apx`, `pages/`, `shared-components/`,
  `supporting-objects/`, `deployments/` — artefatos APEX gerados pelo
  apexlang. NUNCA editar manualmente sem passar pelo fluxo do skill.
- `deployments/default.json` — já existe e aponta para o workspace `ZONA`
  (app id `100`). Não regenerar/sobrescrever sem confirmação explícita do
  usuário.
- `apex-exports/` — backups automáticos em `.sql` que a extensão Oracle SQL
  Developer gera antes de cada Import (ver `apex-exports/README.md`). É
  saída/backup, não fonte de verdade — mesma regra do skill para qualquer
  caminho `apex-exports`: usar só para inspeção/recuperação pontual, nunca
  como base de geração.

Fora de `customers/`, na raiz do repositório (`D:\Repos\APEXlang`):
- [Database/](../Database/) — DDL exportado pelo projeto Database do
  JDeveloper; fonte autoritativa de schema/model. Ver convenção de
  versionamento incremental em [../CLAUDE.md](../CLAUDE.md).
- `specs/` (ainda não existe) — se for usado o fluxo "create app from FR and
  model", os requisitos funcionais (Markdown) devem ficar aqui, na raiz,
  como irmão de `customers/` e `Database/`.
- `.apexlang/application-spec.md` e `.apexlang/app-ux-contract.json` (ainda
  não existem) — nascem na **raiz do repo**, nunca dentro de `customers/` ou
  de `customers/.apex/`. São o plano congelado da aplicação; não apagar.
- `Database/Tables/<TABLE_NAME>.md` — dicionário de dados legível, um
  arquivo por tabela (ex. `Database/Tables/TST_PHONE_TYPES.md`), com colunas
  `Field Name` / `Display Name` / `Description`. Ver regra de autoridade e
  como isso é consumido na seção "Dicionário de dados" abaixo.
- `Prompts/<app>/NNN_NomeDaFuncionalidade.md` — um arquivo por
  página/funcionalidade solicitada (ex.
  `Prompts/customers/001_PhoneTypes.md`), `NNN` sequencial com 3 dígitos por
  subpasta de app. É o registro histórico do pedido; nunca sobrescrever um
  já numerado, sempre criar o próximo número. Funciona como o "checklist
  vivo" leve mencionado na seção de cadência de trabalho — não precisa
  manter isso duplicado em `.apexlang/application-spec.md`.

## Dicionário de dados (`Database/Tables/*.md`)
Cada tabela referenciada num prompt deve ter (ou ganhar, se ainda não
existir) um `Database/Tables/<TABLE_NAME>.md` com: nome da tabela, uma
frase de contexto, e uma tabela markdown `Field Name | Display Name |
Description` cobrindo todas as colunas (inclusive as de auditoria/PK, para
deixar explícito que são ocultas/não editáveis).

- **Autoridade**: mesmo sendo `.md`, isso conta como metadado de
  schema/model autoritativo (não como "requisito em prosa"/hint) — é
  exatamente o que a fonte de precedência do skill chama de "display
  labels" e "semantic facts", que a DDL crua não carrega. Ler sempre os
  dois juntos por tabela: `Database/DDL_vNNN.sql` (estrutura, tipos,
  constraints — verdade estrutural) + `Database/Tables/<TABLE>.md` (label
  de exibição, descrição, dica de campo oculto/não editável). Se algum dia
  os dois conflitarem em um fato estrutural (tipo de dado, nulidade, PK), a
  DDL vence e o conflito deve ser sinalizado, não resolvido em silêncio.
- **Como referenciar num prompt**: não precisa link nem sintaxe especial —
  basta citar o nome da tabela (como já é feito em "Tabela/view:
  TST_PHONE_TYPES" no `Prompts/customers/001_PhoneTypes.md`). Antes de
  gerar qualquer página/região que use essa tabela, ler
  `Database/Tables/<TABLE_NAME>.md` se existir; se não existir, avisar que
  o dicionário está faltando em vez de inventar labels.
- Manter os `Display Name` em português (idioma da UI da aplicação) e
  cobrir 100% das colunas da tabela, mesmo as óbvias — isso é o que vira
  literalmente o `label` dos itens/colunas gerados em relatórios, IGs e
  formulários.

## Raiz de sessão do skill (crítico para todo comando)
O skill resolve contexto local a partir do **diretório de trabalho atual**
(`session_root`), não do diretório onde o script `apexctl.mjs` está
instalado. Portanto:

- Sempre rodar os comandos com o cwd em `D:\Repos\APEXlang` (a raiz do
  repo, não `customers/`, não a pasta do skill).
- Invocar o script pelo caminho absoluto de instalação do skill:
  `C:\Users\Gustavo\.claude\skills\apex\apexlang\tools\apexctl.mjs`.
- **Nunca** `cd` para dentro da pasta do skill e rodar de lá — isso faz o
  probe escanear a documentação do próprio skill como se fosse contexto do
  projeto (README/policies do skill viram "requirements" espúrios).

Validado nesta sessão: rodando de `D:\Repos\APEXlang`, o `workspace probe`
já descobre `Database\DDL_v001.sql` como `data_model` autoritativo e aponta
`customers` como candidato de app pendente de confirmação (ver seção
anterior — já confirmado, não perguntar de novo).

## Conexão com o banco (SQLcl)
- SQLcl instalado em `D:\Bin\sqlcl` (`D:\Bin\sqlcl\bin\sql.exe`), já
  disponível no PATH como `sql`.
- **Ainda não há conexão salva configurada** para este projeto. Antes da
  primeira validação/import ao vivo, criar uma conexão salva (o usuário
  preenche host/porta/service/usuário/senha — nunca colar senha em texto
  puro na conversa):
  ```
  sql /nolog
  SQL> connect -save ZONA_DB -savepwd <usuario>/<senha>@//<host>:<porta>/<service_name>
  ```
  Depois disso, `<db_connection_name>` = `ZONA_DB` (ou o nome escolhido) nos
  comandos abaixo.
- Se houver mais de uma conexão salva, perguntar qual usar — nunca assumir.
- `db_mode = offline` desativa validação/import ao vivo; usar apenas para
  rascunho estrutural sem contato com o banco real.

## Publicar via VS Code (extensão Oracle SQL Developer)
A extensão `oracle.sql-developer` (26.2.1) reconhece `.apx`/`.apex` como
linguagem `apexlang` e adiciona ícones na barra de título do editor quando
um arquivo desse tipo está aberto (ex.: `application.apx`):

- **Import** (ícone de "play") — `sqldeveloper.apexlang.import`: publica o
  APEXlang local no workspace/app conectado.
- **Import File** — importa só o arquivo aberto, não o app inteiro.
- **Pull Diff** — compara o `.apx` local com o app ao vivo no banco.
- **Is Locked** — indicador de lock do app durante import.

Esses comandos só aparecem/habilitam quando há uma **conexão ativa**
selecionada no painel de Connections da extensão (`apexlang.supportsImport`
verdadeiro, `apexlang.usingConnection` controla o estado "rodando"). Ou seja,
"clicar e rodar" funciona, mas só depois de você ter uma conexão configurada
apontando para o schema `ZONA` — a mesma conexão que documentamos na seção
SQLcl acima (a extensão usa o motor do SQLcl por baixo).

**Importante — isso publica só o app, não o schema.** O botão Import
envia a definição da aplicação APEX (páginas, componentes). Ele **não** cria
nem altera tabelas. As tabelas/constraints de `Database/DDL_vNNN.sql`
precisam já existir no schema `ZONA` antes disso — aplicadas separadamente
(deploy do projeto Database do JDeveloper, ou rodando os scripts na ordem
via SQLcl/worksheet). Se uma página referenciar uma tabela que ainda não
existe no schema de destino, o import da página falha ou a página quebra em
runtime mesmo que o `.apx` esteja sintaticamente correto.

Fluxo recomendado antes de usar o botão Import "às cegas": rodar
`apexlang format --strict-structure` e o compiler-truth audit (seção
abaixo) primeiro — o botão Import não faz esse check local, só publica.

## Fluxo de geração (apexlang)
Todos os comandos abaixo assumem cwd = `D:\Repos\APEXlang` e
`<apexctl>` = `C:\Users\Gustavo\.claude\skills\apex\apexlang\tools\apexctl.mjs`.

1. Rodar `node <apexctl> workspace probe` antes de qualquer
   geração/edição com escopo de app.
2. App já existe (`customers/`) — não é caso de "app novo"; não rodar
   `new-app materialize` a menos que se trate de recriar o app do zero com
   confirmação explícita do usuário.
3. Para geração completa a partir de requisitos funcionais + schema: seguir
   `references/workflows/apexlang/workflow-create-app-from-fr-and-model.md`
   do skill, preenchendo `.apexlang/application-spec.md` (raiz do repo,
   template em `references/workflows/apexlang/application-spec.template.md`)
   e `.apexlang/app-ux-contract.json` antes de gerar `.apx` não trivial.
4. Gerar a partir de templates canônicos (`templates/**` do skill), nunca
   inventar sintaxe fora da gramática (`assets/grammar/apexlang.ebnf`).
5. Rodar `node <apexctl> apexlang format --app-path customers --strict-structure`
   e depois o compiler-truth audit
   (`node <apexctl> apexlang compiler-truth audit --app-path customers`)
   antes de considerar qualquer `.apx` pronto.
6. Validação ao vivo:
   `node <apexctl> runtime validate --app-path D:\Repos\APEXlang\customers --db-connection-name <conn> [--apex-root <root>]`
7. Import só acontece após escolha explícita de "Check and import APEXlang
   code" — nunca por inferência automática.

## Cadência de trabalho: uma página/funcionalidade por prompt
Fluxo adotado neste projeto: cada prompt do usuário pede uma página ou uma
funcionalidade pontual (não o app inteiro de uma vez). Isso é compatível com
o skill — o fluxo pesado de `.apexlang/application-spec.md` +
`app-ux-contract.json` é obrigatório só para geração de **app completo** a
partir de FR+modelo (workflow `workflow-create-app-from-fr-and-model.md`);
para incrementos página a página o Claude gera direto um `Generation Plan`
compacto por unidade não trivial, sem precisar da spec completa.

Cuidados por causa dessa granularidade pequena:
- Antes de cada página nova, reconferir o schema atual em `Database/`
  (pode ter crescido desde o último prompt) e o estado atual de
  `shared-components/breadcrumbs.apx`, `lists.apx` (menu de navegação) e
  `authorizations.apx` — toda página de usuário nova precisa de entrada de
  breadcrumb e, se for hub/menu, entrada na lista de navegação. Isso é fácil
  de esquecer quando cada prompt só fala da página em si.
- Se o pedido implicar relação com outra página já existente (ex.: abrir
  detalhe de pedido a partir da lista de clientes), avisar se essa página
  "outra ponta" ainda não existe, em vez de gerar um link morto.
- O registro incremental de páginas/funcionalidades já pedidas vive em
  `Prompts/customers/NNN_*.md` (um arquivo por pedido, nunca sobrescrito) —
  isso já cobre a necessidade de não perder a visão do todo espalhada em
  dezenas de prompts. Não duplicar isso em `.apexlang/application-spec.md`.

## Idioma: pt-BR para o usuário final, inglês para escopo técnico
Regra do projeto: **tudo que é renderizado e visto pelo usuário final vai em
português do Brasil; tudo que é identificador/escopo de desenvolvimento vai
em inglês.** Isso vale para toda geração de `.apx` e para o dicionário de
dados em `Database/Tables/*.md`.

**Inglês (escopo técnico/identificador, nunca visto pelo usuário final):**
- O identificador que vem logo após a palavra-chave da declaração
  (`page 1 (`, `region app-name (`, `entry home (`, `list navigation-bar (`,
  `button create-btn (`, `item p1_customer_id (`, `process save-row (`,
  `dynamicAction refresh-grid (`) — kebab/snake-case em inglês, é a chave
  usada em referências `@algo` e nunca aparece na tela.
- Alias de página (`alias: HOME`), nomes de item (`P1_CUSTOMER_ID`), Static
  ID de região/botão, nomes de Dynamic Action/Process/Validation — seguem a
  convenção técnica do Oracle APEX, sempre em inglês.
- A propriedade `name:` de **página**, **região**, **botão**, **processo**,
  **dynamic action** e **validação** — é o "Name" interno do Page Designer,
  não é exibido ao usuário final na página rodando; pode ficar em inglês
  (ex.: `name: Phone Types`, não precisa traduzir).
- `Field Name` no dicionário `Database/Tables/*.md` — é o nome real da
  coluna no banco, sempre igual ao da DDL (já em inglês por convenção do
  schema).

**Português do Brasil (qualquer coisa vista pelo usuário final):**
- `title:` de página e de região, `label:` de botão/item/lista/breadcrumb,
  textos de mensagens/notificações, texto de validação exibido ao usuário,
  valores de display de LOV, cabeçalhos de coluna derivados do label do
  item.
- `Display Name` e `Description` no dicionário `Database/Tables/*.md` (já é
  a prática adotada).
- Nome de entradas de menu de navegação e de itens de breadcrumb.

**Pegadinha conhecida:** em `entry` de **breadcrumb**
(`shared-components/breadcrumbs.apx`), a propriedade `name:` **não** é o
"Name" interno — é o texto que aparece na trilha de breadcrumb para o
usuário (equivalente ao "Short Name" do APEX nativo). Nesse componente
específico, `name:` segue a regra de português, não a de inglês. Não
generalizar cegamente "propriedade `name` = inglês" sem checar o
componente.

Quando não for óbvio se uma propriedade de um componente específico é
renderizada ao usuário ou é só administrativa, conferir via
`query-valid-props.mjs`/`assets/component-attributes.json` do skill antes
de decidir, em vez de supor pelo nome da propriedade.

**Dívida existente no scaffold:** o app foi gerado com boilerplate padrão
do Universal Theme em inglês (`title: Home` na página 1, `label: Sign Out`
na navigation-bar, `entry home ( name: Home )` no breadcrumb). Isso ainda
não foi corrigido para português — ajustar quando essas páginas/componentes
forem tocados por algum prompt, não é necessário fazer isso preventivamente
sem pedido.

## Convenções de código
- Um bloco/declaração de topo por vez; uma propriedade por linha após a
  declaração de abertura.
- Reutilizar templates apenas quando família/variante do componente,
  contexto pai e modo condicional já baterem exatamente — caso contrário,
  gerar via contrato de gramática, não copiar e adaptar às cegas.
- Nunca deixar `{{...}}` ou placeholders (`SOURCE_TABLE`, `LOOKUP_ID` etc.)
  em artefatos finais.

## Armadilhas conhecidas (achadas na prática, evitar repetir)
Descobertas gerando/testando páginas reais neste app. Valem para qualquer
página nova, não só a que as revelou. Quando um destes casos aparecer de
novo, aplicar direto — não precisa reconsultar compiler-truth do zero.

- **Página de relatório/IG quebra com erro de JS ao abrir** (`Cannot read
  properties of null (reading 'focus')`, `jQuery.Deferred exception`): a
  página não tinha `navigation { cursorFocus: doNotFocusCursor }`. O
  scaffold da Home (`p00001-home.apx`) já fazia isso; toda página cujo
  primeiro elemento não é um campo de texto simples (IG, botões,
  dashboards, hubs) precisa desse bloco explicitamente, senão o APEX tenta
  focar um elemento que não existe no load. Formulários normais (como o
  modal de criação) não precisam — lá o foco automático no primeiro campo é
  desejável.
- **Toda região precisa de `appearance.template` explícito, mesmo
  Interactive Grid** (`Error: Region element not found R<id>` no console,
  vindo de `widget.interactiveGrid.js`): sem `appearance { template: ... }`
  o import deixa a região sem marcação de container e o widget JS não acha
  onde se montar. Não existe "default automático" aceitável — sempre
  emitir. Para saber qual `@/...` usar, conferir
  `customers/shared-components/themes/universal-theme/theme.apx`:
  `componentDefaults.region` e `regionDefaults.<tipo>` listam o template
  padrão real deste tema para cada família de região (ex.: `reports` e o
  default genérico de `region` são `@/standard`; `breadcrumbs` é
  `@/title-bar`; formulário dentro de dialog é
  `dialogDefaults.dialogContentRegion` = `@/blank-with-attributes`). Usar
  esse arquivo como fonte de verdade antes de inventar/copiar um valor de
  exemplo do skill — o exemplo de Interactive Grid do skill usa `@Region`
  como placeholder ilustrativo, não é um valor real para copiar.
- **Região table-backed (`type: interactiveGrid` ou `type: form`) com
  `source.location` + `source.tableName` mas sem `source.type: tableView`**:
  o compilador rejeita `tableName` como "inativo" nesse caso. O template do
  skill (`interactive-grid._common.md`) diz para omitir `type` em grids
  table-backed — está desatualizado nesse ponto; sempre emitir
  `source.type: tableView` junto com `tableName`. Compiler-truth vence o
  template aqui.
- **Coluna `RAW(16)` (ex. PK surrogate)**: não existe `dataType: raw` no
  compilador. Usar `dataType: varchar2`.
- **Coluna oculta (`type: hidden`) de Interactive Grid**: não emitir
  `enableUsersTo { sort: false }` — essa propriedade só é válida quando a
  coluna é `VISIBLE`; em coluna hidden o compilador rejeita.
- **Coluna de Interactive Grid só-leitura dentro de uma região editável**:
  usar `readOnly { type: always }` na coluna (bloco real, confirmado via
  compiler-truth; não aparece no template `interactive-grid._columns._common.md`).
- **`dynamicAction ... action ... ( execution { event: @algo } )`**: a
  propriedade `event` dentro do `execution` de uma `action` aninhada é
  ignorada pelo compilador atual (`Property event cannot be used`), mesmo
  aparecendo em exemplos do skill (`form-page.example.md`,
  `dynamic-actions.refresh-region-after-dialog.md`). Omitir — o evento já
  está definido no bloco `when` pai.
- **Ambiente local**: `apexctl.mjs` chama `python3` internamente; o Windows
  só tinha o stub da Microsoft Store sob esse nome. Corrigido copiando
  `D:\Bin\miniconda3\python.exe` para `python3.exe` na mesma pasta (já no
  PATH). Se sumir/quebrar de novo, é isso que precisa ser refeito.
- **`apexlang format --strict-structure`** não é um gate utilizável neste
  app: acusa "compact structural line expanded" no scaffold inteiro
  (`application.apx`, `theme.apx` etc.), não só em arquivos novos. É dívida
  pré-existente do export original, não um problema introduzido por uma
  página nova — não tratar como bloqueio.
- **`apexlang compiler-truth audit`** pode acusar
  `COMPONENT_AMBIGUOUS column resolves to multiple compiler component
  records (7710, 7930)` em colunas de Interactive Grid mesmo quando estão
  corretas — é limitação da auditoria isolada por arquivo, não do
  conteúdo. Confirmar pelo diagnóstico ao vivo do VS Code (mesmo motor, mais
  contexto) antes de tentar "corrigir" isso.

## Convenções observadas no schema atual (ver [Database/](../Database/))
- PK surrogate `RAW(16)`, preenchida por trigger via `SYS_GUID()` — não
  tornar item obrigatório/editável em formulário de criação.
- `CREATED_AT/BY`, `UPDATED_AT/BY` mantidas por trigger — sempre
  display-only/hidden nos formulários.
- Tabelas com prefixo `TST_` (`TST_CUSTOMERS`, `TST_PHONE_TYPES`,
  `TST_SALES_REPRESENTATIVE`, `TST_CUSTOMER_PHONES`, `TST_ORDERS`).
- Relacionamentos (v001): `TST_CUSTOMER_PHONES` → `TST_CUSTOMERS` e
  `TST_PHONE_TYPES`; `TST_ORDERS` → `TST_CUSTOMERS` e
  `TST_SALES_REPRESENTATIVE`. Todas as FKs `ON DELETE SET NULL`.
- Isso é o estado do `DDL_v001.sql`; sempre reconferir contra a versão mais
  recente em `Database/` antes de gerar código (ver regra de diffs
  incrementais em [../CLAUDE.md](../CLAUDE.md)).

## O que NUNCA fazer
- Não criar diretório `applications/` — o app já vive em `customers/`.
- Não tratar `specs/`/requisitos em prosa como schema autoritativo; schema
  autoritativo vem de `Database/` (todas as versões, em ordem).
- Não rodar `apex import` sem confirmação explícita pós-checagem.
- Não assumir schema = workspace (`ZONA`) sem confirmar.
- Não colar credenciais de banco em texto puro na conversa.

## Comandos úteis
| Ação                          | Comando (cwd = `D:\Repos\APEXlang`) |
| ------------------------------ | ----------------------------------------------------------- |
| Probe do workspace              | `node C:\Users\Gustavo\.claude\skills\apex\apexlang\tools\apexctl.mjs workspace probe` |
| Validar props de componente     | `node C:\Users\Gustavo\.claude\skills\apex\apexlang\tools\query-valid-props.mjs <component>` |
| Formatar/checar estrutura       | `node C:\Users\Gustavo\.claude\skills\apex\apexlang\tools\apexctl.mjs apexlang format --app-path customers --strict-structure` |
| Compiler-truth audit            | `node C:\Users\Gustavo\.claude\skills\apex\apexlang\tools\apexctl.mjs apexlang compiler-truth audit --app-path customers` |
| Validar runtime (live)          | `node C:\Users\Gustavo\.claude\skills\apex\apexlang\tools\apexctl.mjs runtime validate --app-path D:\Repos\APEXlang\customers --db-connection-name <conn> --apex-root <root>` |
