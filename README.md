
<img width="1918" height="200" alt="image" src="https://github.com/user-attachments/assets/e6e12a4f-9e93-4abd-9eda-00f649fca818" />

# Análise Comercial com Qlik Sense
Este projeto tem como objetivo realizar uma análise de dados completa para área comercial utilizando o Qlik Sense, abrangendo desde o processo de ETL, modelagem dos dados até a criação do painel interativo.

O aplicativo oferece uma visão geral das vendas, apresentando métricas essenciais como faturamento, número de vendas, ticket médio, positivação de clientes e análise do mix de produtos. Além disso, conta com abas específicas para uma análise detalhada de clientes e outra dedicada aos produtos.

## Transformação e Modelagem 
<img width="1071" height="481" alt="image" src="https://github.com/user-attachments/assets/6d6930b2-dcc4-4a8c-aab9-8b010616d89a" />

A conexão com os dados é realizada por meio do banco Northwind no PostgreSQL. Utilizando o modelo dimensional Star Schema, as transformações e a criação das tabelas fato e dimensão foram feitas no Editor de Carga do Qlik.

### FATO_VENDAS

````
LIB CONNECT TO 'northwind';

FATO_VENDAS:
LOAD
	DATE(order_date, 'DD-MM-YYYY') AS DATA_VENDA,
	order_id, 
	product_id, 
	MONEY(unit_price) AS PRECO, 
	quantity AS QUANTIDADE, 
	NUM(discount, '##0,00%') AS DESCONTO,
    MONEY((quantity * unit_price) * (1 - discount)) AS VL_FATURAMENTO,
    MONEY((quantity * unit_price) * discount) AS VLR_DESCONTO,
	customer_id;

SELECT "order_date",
	"order_id",
	"product_id",
	"unit_price",
	"quantity",
	"discount",
	"customer_id"
FROM "public"."fato_vendas";
````
### DIM_CALENDARIO
````
TEMP_DATA:
LOAD DISTINCT
    DATA_VENDA
RESIDENT FATO_VENDAS;

DAT_MIN_MAX:
LOAD
    MIN(DATA_VENDA) as MIN_DATA,
    MAX(DATA_VENDA) as MAX_DATA
RESIDENT TEMP_DATA;
DROP TABLE TEMP_DATA;

LET vMinDate = NUM(PEEK('MIN_DATA', 0, 'DAT_MIN_MAX'));
LET vMaxDate = NUM(PEEK('MAX_DATA', 0, 'DAT_MIN_MAX'));

DROP TABLE DAT_MIN_MAX;

TEMP_CALENDARIO:
LOAD
    $(vMinDate) + ITERNO() - 1 AS NUM_DATA
AUTOGENERATE 1
WHILE $(vMinDate) + IterNo() - 1 <= $(vMaxDate);

DIM_CALENDARIO:
LOAD
    DATE(NUM_DATA)             AS DATA_VENDA,
    MONTH(NUM_DATA)            AS MES,
    YEAR(NUM_DATA)             AS ANO,
    MONTHNAME(NUM_DATA)        AS MES_ANO
RESIDENT TEMP_CALENDARIO;
DROP TABLE TEMP_CALENDARIO;
````
### DIM_PRODUTO
````
LIB CONNECT TO 'northwind';

DIM_PRODUTO:
LOAD product_id, 
	product_name, 
	MONEY(unit_price) AS preco, 
	categoria, 
	desc_categoria, 
	fornecedor;

SELECT "product_id",
	"product_name",
	"unit_price",
	"categoria",
	"desc_categoria",
	"fornecedor"
FROM "public"."dim_produto";
````
### DIM_CLIENTE
````
LIB CONNECT TO 'northwind';

DIM_CLIENTE:
LOAD customer_id, 
	company_name AS CLIENTE, 
	city, 
	country;

SELECT "customer_id",
	"company_name",
	"city",
	"country"
FROM "public"."dim_cliente";

````
# Painéis Interativos de Dados

### Visão Geral de Vendas
<img width="1918" height="847" alt="image" src="https://github.com/user-attachments/assets/e7ac40fb-3a1f-4024-9155-2ba8098b4a57" />

### Detalhamento por Cliente
<img width="1918" height="852" alt="image" src="https://github.com/user-attachments/assets/8207e127-3a7c-4e59-af01-f0456d309a8f" />

### Detalhamento por Produto
<img width="1918" height="851" alt="image" src="https://github.com/user-attachments/assets/05344ba2-a410-49ae-99d0-035475048ae6" />
