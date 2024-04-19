Link do dashboard no tableau:
https://public.tableau.com/views/EcommerceBrazil_17108850312770/Painel_Analise?:language=pt-BR&:sid=&:display_count=n&:origin=viz_share_link


# Documentação do Processo ETL (Extract, Transform, Load)

Este repositório contém o código SQL e a documentação associada ao processo de ETL (Extração, Transformação e Load) para o sistema de e-commerce da empresa. O objetivo deste processo é garantir a qualidade e consistência dos dados armazenados no banco de dados, para a exibição no Tableau.

## Visão Geral

O processo ETL é composto por várias etapas, cada uma das quais é descrita abaixo:

Extração (Extract): Durante esta etapa, os dados foram extraídos do banco de dados do sistema de e-commerce. São selecionadas diversas tabelas que contêm informações essenciais sobre clientes, pedidos, produtos, pagamentos e etc.

Transformação (Transform): Nesta etapa, os dados extraídos são processados e transformados para garantir sua consistência e qualidade. Isso inclui a limpeza de dados, detecção e tratamento de duplicatas, normalização de formatos e cálculos de métricas adicionais.

Load: Por fim, os dados transformados são carregados de volta ao banco de dados, substituindo ou atualizando as informações existentes. Aqui os dados já estão prontos para serem consumidos no Tableau.


## Estrutura do Repositório

SQL_File.sql: Este arquivo contém o código SQL utilizado para realizar o processo de ETL. Ele está organizado em seções que correspondem a cada etapa do processo, incluindo consultas para verificação e limpeza dos dados, bem como consultas analíticas para extrair insights.
README.md: Este arquivo contém a documentação detalhada do processo de ETL. Aqui você encontrará uma visão geral do processo, bem como instruções sobre como executar o código SQL e interpretar os resultados.




# Segue Código SQL:



use EcommerceBrazil

/*

Verificando e limpando os dados

*/

-- # 1.0 Verificando a tabela CUSTOMERS #

-- 1.0.1 Verificando os dados da coluna CUSTOMER_STATE
SELECT TOP (1000)     
	[customer_state],
	count(distinct customer_id) as contagem
FROM 
	[EcommerceBrazil].[dbo].[customers]
group by 
	customer_state

-- 1.0.2 Verificando duplicidade de customers_unique_id

SELECT customer_unique_id, COUNT(*)
FROM [customers]
GROUP BY customer_unique_id
HAVING COUNT(*) > 1;

-- 1.0.3 A coluna customers_unique_id apresentam alguns repetidos, vale a pena remove-los?

SELECT *
FROM [customers]
where customer_unique_id = '63cfc61cee11cbe306bff5857d00bfe4'

/*

Conclusão:
A tabela de customres apresenta dados consistentes. Alguns CUSTOMER_UNIQUE_ID se repetem porém apresentam CUSTOMER_ID diferentes, então não irei remover os CUSTOMER_UNIQUE_ID duplicados. 

*/


-- 2.0 Verificando a tabela geolocation

SELECT top 100 *
FROM geolocation

SELECT geolocation_state, count(distinct(geolocation_state)) estado
FROM geolocation
group by geolocation_state

-- 2.0.1 Alguns dados da coluna geolocation_state estão com um espaçamento. Irei remove-los

SELECT geolocation_state, count(distinct(geolocation_state)) estado
FROM geolocation
where geolocation_state like '% %' 
group by geolocation_state

-- 2.0.2 Atualizando a tabela com os dados sem o espaçamento.
update geolocation
set geolocation_state = replace(geolocation_state,' ', '')



-- # 3.0 Verificando a tabela order_items

select top 100 *
from order_items


-- 3.0.1 Alguns pedidos estão repetidos, irei verificar o status das compras.

SELECT 
    oi.order_id, 
    oi.order_item_id, 
    oi.product_id, 
    oi.price,
    o.order_approved_at, 
    o.order_delivered_customer_date, 
    o.order_status
FROM 
    order_items AS oi
JOIN 
    orders AS o 
ON 
    oi.order_id = o.order_id
WHERE 
    oi.order_id IN (
        SELECT 
            order_id
        FROM 
            order_items
        GROUP BY
            order_id
        HAVING
            COUNT(order_id) > 1
    )
ORDER BY 
    oi.order_id, 
    oi.order_item_id;


/*

Conclusão: Uma order_id pode ter varios product_id iguais, logo, os dados não estão repetidos. São produtos iguais feitos na mesma compra.

*/

-- # 4.0 Verificando a tabela order_paymentes

select top 100 *
from order_payments

-- 4.0.1 Removendo os "_" da coluna payment_type

select 
	payment_type
from 
	order_payments
where payment_type like '%_%' 

update
	order_payments
set 
	payment_type = REPLACE(payment_type, '_', ' ')

/*

Conclusão: As colunas da tabela de order_payments estão bem consistentes, não havendo necessidade de manipulações, apenas substitui os "_" da coluna payment_type.

*/

-- #############################################

/*

Perguntas:


*/

-- 01. Qual é a média de tempo de entrega dos pedidos por estado dos clientes?

SELECT 
	c.customer_state, 
	AVG(DATEDIFF(day, o.order_approved_at, o.order_delivered_customer_date)) AS Media_Entrega
FROM 
	orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE 
	o.order_status = 'delivered'
GROUP BY 
	c.customer_state;

-- 02. Quais são os produtos com as melhores avaliações médias?

SELECT 
	p.product_id, 
    p.product_category_name, 
    AVG(CAST(r.review_score AS FLOAT)) AS avg_review_score
FROM 
    order_items oi
    JOIN order_reviews r ON oi.order_id = r.order_id
    JOIN products p ON oi.product_id = p.product_id
GROUP BY 
    p.product_id, 
    p.product_category_name
ORDER BY 
    avg_review_score DESC



-- 03. Total de pedidos cancelados?

SELECT
    order_status,
    COUNT(o.order_id) AS contagem
FROM
    orders AS o
WHERE
    purchase_date >= '2018-01-01' 
GROUP BY
    order_status;



-- 04. Total de venda por Estado

SELECT 
	c.customer_state, 
	round(sum(cast(oi.price as float)) + sum(cast(oi.freight_value as float)),2) as Total_venda
FROM 
	orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN customers as c ON o.customer_id = c.customer_id
GROUP BY 
	c.customer_state
ORDER BY 
	Total_venda desc;

-- 05. Formas de pagamento

SELECT
	payment_type,
	count(payment_type) as Total
FROM
	order_payments
GROUP BY
	payment_type
ORDER BY 
	Total desc
