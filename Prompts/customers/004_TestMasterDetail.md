Quero uma página de teste isolada, só para validar o relacionamento nativo
de Interactive Grid mestre-detalhe do Oracle APEX antes de reaplicá-lo na
página de Clientes (`p00006-customers.apx`). Lá, duas tentativas já
falharam de formas diferentes:

1. Configurando `masterRegion` na região filha **e** `masterColumn` na
   coluna de FK: o import passou na validação (`runtime validate`), mas em
   runtime a própria IG mestre quebrou ao carregar, com o erro no console
   `Error: cannot call methods on interactiveGrid prior to initialization;
   attempted to call method 'getCurrentViewId'`.
2. Configurando só `masterRegion` (sem `masterColumn` na coluna, supondo
   que o APEX detectasse a FK sozinho): o Page Designer acusou `Interactive
   Grid "Phones Grid" doesn't have a master column defined which is
   required for a master detail relationship.`

Ou seja, `masterColumn` **é** obrigatório, mas configurá-lo do jeito que
tentei antes (`masterDetail { masterColumn: @ID }`, resolvendo `@ID` no
escopo da região mestre) quebrou o carregamento da IG mestre. Preciso
entender a sintaxe/config correta antes de tentar de novo na página real.

Esta página de teste deve ficar isolada (sem abas, sem modal, sem outras
regiões, sem entrada de menu/breadcrumb) para eu conseguir isolar só a
variável do relacionamento mestre-detalhe.

Fonte de dados (reaproveitar tabelas já existentes no app, que já têm FK
real entre si — `TST_CUSTOMER_PHONES_FK2`):
- Região 1 (mestre): tabela `TST_CUSTOMERS`, chave primária `ID`. Colunas
  exibidas: `TAX_ID`, `NICKNAME` (somente leitura, não precisa permitir
  edição nessa página de teste).
- Região 2 (detalhe): tabela `TST_CUSTOMER_PHONES`, chave primária `ID`,
  FK `CUSTOMER_ID` → `TST_CUSTOMERS.ID`. Colunas exibidas e editáveis:
  `PHONE_TYPE_ID` (LOV com `TST_PHONE_TYPES.CODE`), `DDI`, `DDD`,
  `PHONE_NUMBER`, `NOTES`. Operações permitidas: `add`, `update`, `delete`
  (CRUD completo).

Página:
- Duas regiões lado a lado ou uma embaixo da outra (o layout exato não
  importa nessa página de teste) — Região 1 com a IG mestre de clientes,
  Região 2 com a IG detalhe de telefones.
- O relacionamento entre as duas IGs deve usar o recurso **nativo** de
  Interactive Grid Master Detail do Oracle APEX (o mesmo que aparece no
  Page Designer em Region → Attributes → "Master Detail" → "Master
  Region", e na coluna de FK → "Master Detail" → "Master Column") — não
  usar filtro manual via `sqlQuery` + item de página + dynamic action de
  refresh (isso já foi testado e funciona, mas é o que estou tentando
  substituir).

Antes de implementar, pesquisar a documentação oficial do Oracle APEX
sobre Interactive Grid Master Detail (não só inferir pela gramática do
compilador APEXlang) para confirmar:
- Quais atributos são realmente obrigatórios na região filha e na coluna
  de FK, e o valor/formato exato esperado para cada um.
- Se há algum pré-requisito na região mestre (ex.: modo de seleção de
  linha, Static ID, etc.) para a relação funcionar sem quebrar a
  inicialização do widget JS.
- Se há alguma restrição de ordem de renderização entre mestre e detalhe
  na página.

Critérios de aceite (testar após importar, no navegador):
- Selecionar uma linha na IG de clientes filtra automaticamente a IG de
  telefones para aquele cliente, sem precisar de dynamic action manual.
- Clicar em "Add Row" na IG de telefones, preencher os campos e salvar
  grava o registro com `CUSTOMER_ID` preenchido automaticamente (sem
  `ORA-01400: cannot insert NULL`).
- Nenhum erro no console do navegador ao carregar a página nem ao trocar
  a seleção do cliente.
- `apexlang compiler-truth audit` e `runtime validate` passam limpos.

Outros detalhes:
- Deve ser utilizada como ID da página o próximo ID livre mas não depois
  da página de login.
- Não criar entrada de menu nem de breadcrumb para esta página — é só
  para teste/diagnóstico, acesso direto pela URL/Page Designer.

## Implementação
(preencher depois de implementado)

## Observações da implementação
(preencher depois de implementado, incluindo a config exata que funcionou
e por quê — esse resultado deve virar uma nova entrada em "Armadilhas
conhecidas" no `customers/CLAUDE.md` antes de reaplicar na página 6)
