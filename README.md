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
Instruções de Uso
