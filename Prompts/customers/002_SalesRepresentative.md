Quero uma página de para realizar a manutenção do cadastro de Representantes de Vendas no app customers.

Fonte de dados:
- Tabela/view: TST_SALES_REPRESENTATIVE
- Chave primária: ID
- Colunas exibidas: EMPLOYEE_CODE, TAX_ID, NICKNAME, FULL_NAME, EMAIL, CELL_PHONE
- Colunas editáveis: FULL_NAME, EMAIL, CELL_PHONE

Região principal:
- Tipo: Interactive Grid
- Operações permitidas no grid: apenas UPDATE (desabilitar insert/delete inline)

Criação de registro:
- Deve ocorrer via modal (não pelo próprio IG)
- Botão de disparo:  "Novo Representante de Vendas" no topo da região
- Campos do formulário modal:EMPLOYEE_CODE, TAX_ID, NICKNAME, FULL_NAME, EMAIL, CELL_PHONE sendo obrigatório EMPLOYEE_CODE, TAX_ID, NICKNAME e EMAIL
- No formulário os campos EMPLOYEE_CODE, TAX_ID, NICKNAME devem estar na mesma linha
- Após salvar: fechar modal e atualizar o grid 

Menu:
- Criar uma nova entrada de menu chamada "Time de Vendas"
- Posição/categoria: Dentro do menu principal chamado "Parametrização"
- Ícone: encontrar o icone mais aderente
- Página de destino: esta nova página

Outros detalhes:
- Deve ser utilizada como ID da página o próximo ID livre mas não depois da página de login.

## Implementação
- Página 4 (`p00004-sales-representative.apx`) + modal de criação página 5
  (`p00005-sales-representative-create.apx`).
- Entrada de breadcrumb e de menu ("Parametrização" > "Time de Vendas")
  criadas em `shared-components/`.

## Atenção a erros já conhecidos
Ao abrir a página 2 pela primeira vez no runtime, o console acusava:
```
jQuery.Deferred exception: Cannot read properties of null (reading 'focus')
```
e a página não terminava de carregar. Causa: faltava
`navigation { cursorFocus: doNotFocusCursor }` na página — sem isso o APEX
tenta focar automaticamente o "primeiro item" no load, e numa página de
Interactive Grid/botões (sem um campo de texto simples como primeiro
elemento) esse alvo não existe. Corrigido na própria página; regra geral
registrada em `customers/CLAUDE.md` ("Armadilhas conhecidas") para toda
página nova de relatório/IG/hub seguir por padrão.

Em seguida, novo erro no console ao abrir a página:
```
Error: Region element not found R9206114568513568
    at widget.interactiveGrid.js ...
```
Causa: a região do Interactive Grid (`phone-types-grid`) tinha sido gerada
sem `appearance { template: ... }`. Toda região do APEX precisa de um
template explícito; sem ele o container HTML não é montado e o widget JS
do Interactive Grid não encontra onde se inicializar. Corrigido adicionando
`appearance { template: @/standard templateOptions: #DEFAULT# }`, valor
confirmado em `customers/shared-components/themes/universal-theme/theme.apx`
(`componentDefaults.region` e `regionDefaults.reports` = `@/standard` neste
tema). Regra geral também registrada em `customers/CLAUDE.md`.

Atenção na implementação dessa nova funcionaludade.

## Observações da implementação
- Todas as armadilhas conhecidas foram aplicadas preventivamente
  (`navigation.cursorFocus`, `appearance.template` explícito no IG,
  `source.type: tableView`) — nenhum erro de JS novo apareceu ao gerar esta
  página.
- **Desvio do pedido**: `CELL_PHONE` não estava na lista de campos
  obrigatórios do modal, mas a coluna é `NOT NULL` no
  `Database/DDL_v001.sql`. Deixei como obrigatório no formulário (mesma
  regra de `../../CLAUDE.md`: constraint do banco vence sobre a lista do
  prompt) — senão salvar sem preencher geraria erro de constraint do Oracle
  em vez de uma validação amigável do APEX. `TAX_ID` foi mantido
  obrigatório como pedido, mesmo sendo opcional no banco (regra de negócio
  mais restritiva que a estrutural, sem conflito).
- Layout "mesma linha" de `EMPLOYEE_CODE`/`TAX_ID`/`NICKNAME` no modal
  implementado com `layout.columnSpan: 4` nos três + `layout.startNewRow:
  false` nos dois últimos (propriedade confirmada via compiler-truth,
  `pageItem.layout.startNewRow`).
- Ícone do menu: `fa-users` (mais aderente a "Time de Vendas" do que um
  ícone de pessoa única).
