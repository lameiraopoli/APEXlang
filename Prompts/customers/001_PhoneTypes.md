Quero uma página de para realizar a manutenção do cadastro de tipos de telefone no app customers.

Fonte de dados:
- Tabela/view: TST_PHONE_TYPES
- Chave primária: ID
- Colunas exibidas: CODE, NAME, DESCRIPTION
- Colunas editáveis: NAME, DESCRIPTION

Região principal:
- Tipo: Interactive Grid
- Operações permitidas no grid: apenas UPDATE (desabilitar insert/delete inline)

Criação de registro:
- Deve ocorrer via modal (não pelo próprio IG)
- Botão de disparo:  "Criar Novo Tipo de Telefone" no topo da região
- Campos do formulário modal: CODE, NAME, DESCRIPTION sendo obrigatório CODE e NAME
- Após salvar: fechar modal e atualizar o grid 

Menu:
- Criar uma nova entrada de menu chamada "Tipos de Telfone"
- Posição/categoria: Dentro do menu principal chamado "Parametrização"
- Ícone: encontrar o icone mais aderente
- Página de destino: esta nova página

Outros detalhes:
- Deve ser utilizada como ID da página o próximo ID livre mas não depois da página de login.

## Implementação
- Página 2 (`p00002-phone-types.apx`) + modal de criação página 3
  (`p00003-phone-types-create.apx`).
- Entrada de breadcrumb e de menu ("Parametrização" > "Tipos de Telefone")
  criadas em `shared-components/`.

## Problema encontrado pós-import e correção
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

