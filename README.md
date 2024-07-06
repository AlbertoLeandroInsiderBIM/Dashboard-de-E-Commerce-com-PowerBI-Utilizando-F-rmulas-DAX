
## Projeto de Modelagem de Dados - Star Schema
Apresentação Pessoal
Olá, meu nome é Alberto Leandro, sou arquiteto e urbanista especialista em ciência de dados. Aqui explicarei o processo de criação de um modelo de dados baseado no esquema estrela utilizando a planilha "Financials_origem".

## Descrição do Projeto
Este projeto tem como objetivo transformar a tabela única "Financial Sample" em tabelas dimensão e fato, criando um modelo de dados baseado no star schema. Esse modelo facilita a análise de dados e a geração de insights.

## Estrutura do Projeto

### Tabela Original
Financials_origem: Esta tabela é a origem dos dados e será utilizada como backup (modo oculto) para a criação das novas tabelas dimensão e fato.

### Tabelas Dimensão
D_Produtos

Colunas: ID_produto, Produto, Média de Unidades Vendidas, Médias do valor de vendas, Mediana do valor de vendas, Valor máximo de Venda, Valor mínimo de Venda
D_Produtos_Detalhes

Colunas: ID_produto, Discount Band, Sale Price, Units Sold, Manufacturing Price
D_Descontos

Colunas: ID_produto, Discount, Discount Band
D_Detalhes

Colunas: Informações adicionais sobre vendas que não foram contempladas nas demais tabelas dimensão.
D_Calendário

Criada utilizando a função DAX calendar()
Tabela Fato
F_Vendas
Colunas: SK_ID, ID_Produto, Produto, Units Sold, Sales Price, Discount Band, Segment, Country, Sellers, Profit, Date

## Processo de Construção do Diagrama

### Passo 1: Criação das Tabelas Dimensão

-- 01 D_Produtos: Criada a partir da tabela original, agrupando as informações de produtos e calculando métricas como média, mediana, valor máximo e mínimo de vendas.
CREATE TABLE D_Produtos AS

```sql
SELECT
    ID_produto,
    Produto,
    AVG(Units_Sold) AS Media_Unidades_Vendidas,
    AVG(Sales_Amount) AS Media_Valor_Vendas,
    MEDIAN(Sales_Amount) AS Mediana_Valor_Vendas,
    MAX(Sales_Amount) AS Valor_Maximo_Venda,
    MIN(Sales_Amount) AS Valor_Minimo_Venda
FROM
    Financials_origem
GROUP BY
    ID_produto, Produto;    
    
-- 02 D_Produtos_Detalhes: Extração dos detalhes dos produtos, incluindo faixa de desconto, preço de venda, unidades vendidas e preço de fabricação.

CREATE TABLE D_Produtos_Detalhes AS
SELECT
    ID_produto,
    Discount_Band,
    Sale_Price,
    Units_Sold,
    Manufacturing_Price
FROM
    Financials_origem;

--D_Descontos: Criação da tabela de descontos com base nos dados de produtos e faixas de desconto.

CREATE TABLE D_Descontos AS
SELECT
    ID_produto,
    Discount,
    Discount_Band
FROM
    Financials_origem;

--D_Detalhes: Coletando informações adicionais sobre as vendas que não foram incluídas nas outras tabelas dimensão.

CREATE TABLE D_Detalhes AS
SELECT
    DISTINCT Segment, Country, Sellers
FROM
    Financials_origem;

--D_Calendário: Gerada utilizando a função DAX calendar() para criar uma tabela de datas abrangente.

D_Calendário = CALENDAR(
    MIN(Financials_origem[Date]),
    MAX(Financials_origem[Date])
)

Passo 2: Criação da Tabela Fato

--F_Vendas: Consolidando os dados de vendas com chaves estrangeiras (ID_Produto) que se relacionam com as tabelas dimensão, além de incluir métricas como unidades vendidas, preço de venda, faixa de desconto, segmento, país, vendedores, lucro e data.

CREATE TABLE F_Vendas AS
SELECT
    NEWID() AS SK_ID,
    ID_produto,
    Produto,
    Units_Sold,
    Sales_Price,
    Discount_Band,
    Segment,
    Country,
    Sellers,
    Profit,
    Date
FROM
    Financials_origem;

Passo 3: Ajustes e Reorganização das Colunas

--Reorganização das colunas das tabelas para garantir uma estrutura lógica e otimizada para análise de dados.

ALTER TABLE F_Vendas
REORDER COLUMNS (
    SK_ID, 
    ID_produto, 
    Produto, 
    Units_Sold, 
    Sales_Price, 
    Discount_Band, 
    Segment, 
    Country, 
    Sellers, 
    Profit, 
    Date
);
 
