# TST_ORDERS

Tabela com as ordens de vendas que um determinado cliente possui. Uma ordem de venda é feita para um cliente por um representante de vendas

# Campos

|Field Name|Display Name|Description|
|--|--|--|
|`ID`|`-`|PK da tabela, campo não exibido|
|`CUSTOMER_ID`|Cliente|FK para tabela de cliente TST_CUSTOMERS|
|`SALES_REPRESENTATIVE_ID`|Vendedor|FK para tabela de time de vendas TST_SALES_REPRESENTATIVE|
|`ORDER_NRO`|Nro Pedido|Número do pedido de venda|
|`ORDER_DATE`|Data Pedido|Data em que o pedido de venda foi registrado|
|`ORDER_VALUE`|Valor Pedido|Valor total do pedido de venda|
|`PAID_AMOUNT`|Valor Pago|Valor já pago pelo cliente|
|`COMMISSION_RATE`|Comissão (%)|% da comissão da venda desse pedido|
|`NOTES`|Notas|Texto livre para comentários sobre o pedido|
|`CREATED_AT`|Criado em|Data e hora da criação do registro|
|`CREATED_BY`|Criado por|Nome do usuário responsável pela criação|
|`UPDATED_AT`|Alterado em|Data e hora da última alteração|
|`UPDATED_BY`|Alterado por|Nome do usuário responsável pela última alteração|








