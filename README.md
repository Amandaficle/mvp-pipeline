# MVP --- Construção de um Pipeline de Dados na Nuvem

## 1. Contexto de Negócios e Perguntas

### 1.1 A base utilizada

Este MVP foi desenvolvido a partir da **Brazilian E-Commerce Public
Dataset by Olist**, uma base pública disponibilizada no Kaggle pela
Olist. O conjunto reúne aproximadamente 100 mil pedidos realizados entre
2016 e 2018 em diferentes marketplaces no Brasil e foi construído a
partir de dados comerciais reais, posteriormente anonimizados.

A base é composta por nove arquivos relacionados entre si, abrangendo
diferentes perspectivas da operação de comércio eletrônico:

-   clientes e sua localização;
-   pedidos e seus respectivos status e datas;
-   itens dos pedidos;
-   pagamentos;
-   avaliações dos clientes;
-   produtos e categorias;
-   vendedores;
-   dados de geolocalização;
-   tradução dos nomes das categorias de produtos.

Essa estrutura relacional é adequada para um projeto de Engenharia de
Dados porque uma única pergunta de negócio frequentemente depende da
combinação de diferentes fontes. Por exemplo, compreender o desempenho
das entregas exige relacionar pedidos, clientes e datas; analisar
faturamento por categoria exige combinar itens vendidos com informações
de produtos; e relacionar experiência do cliente à logística exige
conectar entregas e avaliações.

Os arquivos originais utilizados neste projeto são os mesmos componentes
disponibilizados na versão pública do dataset, incluindo
`olist_orders_dataset.csv`, `olist_order_items_dataset.csv`,
`olist_products_dataset.csv`, `olist_customers_dataset.csv`,
`olist_order_reviews_dataset.csv`, `olist_sellers_dataset.csv`,
`olist_order_payments_dataset.csv`, `olist_geolocation_dataset.csv` e
`product_category_name_translation.csv`.

### 1.2 Por que escolhi essa base ?

A escolha da base foi orientada pela possibilidade de construir um problema
de negócio suficientemente próximo de um cenário real e, ao mesmo tempo, adequado ao contexto da disciplina.

Me interessei pela base pois uma operação de comércio eletrônico reúne diferentes dimensões que precisam 
ser analisadas de forma integrada: vendas, produtos, clientes, localização, logística e experiência
após a compra. Isso permite exercitar não apenas a manipulação de dados, mas também decisões de modelagem, integração entre tabelas, 
tratamento de qualidade e construção de estruturas analíticas orientadas a perguntas de negócio.

### 1.3 Contexto do problema de negócio

O problema deste MVP consiste em utilizar os dados para construir uma
visão integrada do desempenho comercial e operacional do e-commerce.

A análise procura responder a cinco dimensões complementares desse
problema:

1.  quais categorias apresentam maior participação no faturamento;
2.  como o volume de pedidos e o faturamento se distribuem
    territorialmente;
3.  se atrasos na entrega estão associados a avaliações menores dos
    clientes;
4.  como o custo de frete varia entre diferentes contextos de operação;
5.  como o volume de pedidos, o faturamento e o ticket médio evoluem ao
    longo do tempo.

Essas perguntas foram escolhidas porque representam questões que
poderiam surgir em uma área real de negócio de um marketplace ou
operação de comércio eletrônico. Elas combinam indicadores comerciais,
geográficos, logísticos e de experiência do cliente, permitindo observar
o negócio por diferentes perspectivas.

A estrutura do pipeline foi definida considerando o uso final dos dados.
As tabelas Gold construídas nas etapas anteriores foram organizadas para
facilitar justamente as análises apresentadas neste notebook.

### 1.4 Perguntas de negócio

**Q1 --- Categorias:** Quais categorias concentram o maior faturamento?

**Q2 --- Distribuição geográfica:** Quais estados concentram maior
volume e faturamento?

**Q3 --- Entrega e satisfação:** Pedidos atrasados apresentam avaliações
menores?

**Q4 --- Frete:** Como o frete varia por estado e categoria?

**Q5 --- Evolução temporal:** Como pedidos, faturamento e ticket médio
evoluíram ao longo do tempo?

### 1.5 Fonte e licença dos dados

Os dados utilizados são provenientes do **Brazilian E-Commerce Public
Dataset by Olist**, disponibilizado publicamente no Kaggle.

Fonte: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

A licença e os termos de utilização devem ser consultados diretamente na
página oficial do dataset. Os dados de origem não são redistribuídos
neste repositório.

------------------------------------------------------------------------

## 2. Carga dos Dados

A carga dos dados foi realizada no ambiente **Databricks**.

Os nove arquivos disponibilizados pela fonte foram inicialmente
carregados em sua estrutura original para a camada **Bronze**. Essa
etapa teve como objetivo preservar uma representação próxima à fonte
antes da aplicação das transformações realizadas nas etapas posteriores.

A carga foi desenvolvida no notebook **01_Bronze**.

O notebook realiza a leitura dos arquivos, verificações iniciais de
estrutura e volume e a persistência das tabelas na plataforma.

### Evidência 1 --- carga dos dados

<img width="1083" height="571" alt="image" src="https://github.com/user-attachments/assets/2ed3e592-ea5e-4bc3-9cee-f5b49637906c" />


### Evidência 2 --- tabelas Bronze

<img width="822" height="464" alt="image" src="https://github.com/user-attachments/assets/290c13d1-8294-437a-95a5-de28e5d98f28" />


------------------------------------------------------------------------

## 3. Modelagem e Catálogo de Dados

O pipeline foi organizado segundo uma arquitetura de camadas:

**Bronze → Silver → Gold**

A lógica adotada separa os dados de acordo com seu nível de
transformação e finalidade.

### Evidência 3 --- catálogo de dados no Databricks (print de consulta feita na plataforma seguido do detalhamento)
<img width="1250" height="680" alt="image" src="https://github.com/user-attachments/assets/61ab9270-266a-47b8-b8a1-713cf9667f41" />

### 3.1 Camada Bronze

A camada Bronze mantém os dados próximos à estrutura original da fonte.

Foram criadas as seguintes tabelas:

  Tabela
  -------------------------------
  `bronze_customers`
  `bronze_geolocation`
  `bronze_order_items`
  `bronze_order_payments`
  `bronze_order_reviews`
  `bronze_orders`
  `bronze_products`
  `bronze_sellers`
  `bronze_category_translation`

### 3.2 Camada Silver

A camada Silver concentra os dados tratados e padronizados.

Entre os principais tratamentos realizados estão:

-   conversão e padronização de tipos;
-   tratamento de valores inválidos;
-   tratamento de duplicidades;
-   tratamento de datas;
-   padronização de informações textuais;
-   tratamento de chaves;
-   criação de indicadores derivados;
-   preparação dos dados para modelagem analítica.

Na tabela de avaliações, a deduplicação foi realizada considerando a
combinação `review_id + order_id`, evitando utilizar apenas `order_id`
como chave, uma vez que um pedido pode possuir mais de uma avaliação.

Também foram tratados valores inválidos de `review_score`, mantendo como
válidas as notas dentro da escala de 1 a 5.

### 3.3 Camada Gold

A camada Gold foi construída para disponibilizar dados preparados para
consumo analítico.

#### Dimensões

  Tabela                Finalidade
  --------------------- -------------------------------
  `gold_dim_customer`   Caracterização dos clientes
  `gold_dim_product`    Caracterização dos produtos
  `gold_dim_seller`     Caracterização dos vendedores
  `gold_dim_date`       Dimensão temporal

#### Fatos

  Tabela                         Finalidade
  ------------------------------ -----------------------------------
  `gold_fact_sales_items`        Análise dos itens vendidos
  `gold_fact_orders`             Análise dos pedidos
  `gold_fact_delivery_reviews`   Relação entre entrega e avaliação

#### Tabelas analíticas

  Tabela                       Pergunta respondida
  ---------------------------- ----------------------------------
  `gold_q1_category_revenue`   Q1 --- faturamento por categoria
  `gold_q2_state_sales`        Q2 --- vendas por estado
  `gold_q3_delivery_review`    Q3 --- entrega e avaliação
  `gold_q4_freight`            Q4 --- frete
  `gold_q5_temporal`           Q5 --- evolução temporal

### 3.4 Catálogo de Dados

1. Camada Bronze

A camada Bronze é formada pelos dados de origem carregados diretamente dos arquivos CSV. Nessa etapa não são aplicadas regras de negócio ou transformações analíticas.

####bronze_customers

Finalidade: armazenar os dados cadastrais e de localização dos clientes.

| Campo                      | Tipo    | Domínio / significado              | Origem / transformação |
| -------------------------- | ------- | ---------------------------------- | ---------------------- |
| `customer_id`              | string  | Identificador do cliente no pedido | CSV de origem          |
| `customer_unique_id`       | string  | Identificador único do cliente     | CSV de origem          |
| `customer_zip_code_prefix` | integer | Prefixo do CEP                     | CSV de origem          |
| `customer_city`            | string  | Município do cliente               | CSV de origem          |
| `customer_state`           | string  | UF do cliente                      | CSV de origem          |


#### bronze_geolocation

Finalidade: armazenar informações geográficas associadas aos prefixos de CEP.

| Campo                         | Tipo    | Domínio / significado | Origem / transformação |
| ----------------------------- | ------- | --------------------- | ---------------------- |
| `geolocation_zip_code_prefix` | integer | Prefixo do CEP        | CSV de origem          |
| `geolocation_lat`             | double  | Latitude geográfica   | CSV de origem          |
| `geolocation_lng`             | double  | Longitude geográfica  | CSV de origem          |
| `geolocation_city`            | string  | Município             | CSV de origem          |
| `geolocation_state`           | string  | UF                    | CSV de origem          |


#### bronze_order_items

Finalidade: armazenar os itens que compõem cada pedido.

| Campo                 | Tipo      | Domínio / significado                             | Origem / transformação |
| --------------------- | --------- | ------------------------------------------------- | ---------------------- |
| `order_id`            | string    | Identificador do pedido                           | CSV de origem          |
| `order_item_id`       | integer   | Identificador sequencial do item dentro do pedido | CSV de origem          |
| `product_id`          | string    | Identificador do produto                          | CSV de origem          |
| `seller_id`           | string    | Identificador do vendedor                         | CSV de origem          |
| `shipping_limit_date` | timestamp | Data/hora limite para envio                       | CSV de origem          |
| `price`               | double    | Valor do item                                     | CSV de origem          |
| `freight_value`       | double    | Valor do frete do item                            | CSV de origem          |


#### bronze_order_payments

Finalidade: armazenar os pagamentos associados aos pedidos.

| Campo                  | Tipo    | Domínio / significado            | Origem / transformação |
| ---------------------- | ------- | -------------------------------- | ---------------------- |
| `order_id`             | string  | Identificador do pedido          | CSV de origem          |
| `payment_sequential`   | integer | Sequência do pagamento no pedido | CSV de origem          |
| `payment_type`         | string  | Modalidade de pagamento          | CSV de origem          |
| `payment_installments` | integer | Número de parcelas               | CSV de origem          |
| `payment_value`        | double  | Valor do pagamento               | CSV de origem          |


#### bronze_order_reviews

Finalidade: armazenar as avaliações e comentários associados aos pedidos.

| Campo                     | Tipo   | Domínio / significado                                          | Origem / transformação |
| ------------------------- | ------ | -------------------------------------------------------------- | ---------------------- |
| `review_id`               | string | Identificador da avaliação                                     | CSV de origem          |
| `order_id`                | string | Identificador do pedido                                        | CSV de origem          |
| `review_score`            | string | Nota da avaliação, posteriormente tratada como escala de 1 a 5 | CSV de origem          |
| `review_comment_title`    | string | Título do comentário                                           | CSV de origem          |
| `review_comment_message`  | string | Texto do comentário                                            | CSV de origem          |
| `review_creation_date`    | string | Data de criação da avaliação                                   | CSV de origem          |
| `review_answer_timestamp` | string | Data/hora da resposta                                          | CSV de origem          |


#### bronze_orders

Finalidade: armazenar os dados gerais dos pedidos e seus eventos de compra e entrega.

| Campo                           | Tipo      | Domínio / significado                 | Origem / transformação |
| ------------------------------- | --------- | ------------------------------------- | ---------------------- |
| `order_id`                      | string    | Identificador do pedido               | CSV de origem          |
| `customer_id`                   | string    | Identificador do cliente              | CSV de origem          |
| `order_status`                  | string    | Situação do pedido                    | CSV de origem          |
| `order_purchase_timestamp`      | timestamp | Data/hora da compra                   | CSV de origem          |
| `order_approved_at`             | timestamp | Data/hora da aprovação                | CSV de origem          |
| `order_delivered_carrier_date`  | timestamp | Data/hora de entrega à transportadora | CSV de origem          |
| `order_delivered_customer_date` | timestamp | Data/hora de entrega ao cliente       | CSV de origem          |
| `order_estimated_delivery_date` | timestamp | Data estimada de entrega              | CSV de origem          |


#### bronze_products

Finalidade: armazenar o cadastro dos produtos.
| Campo                        | Tipo    | Domínio / significado      | Origem / transformação |
| ---------------------------- | ------- | -------------------------- | ---------------------- |
| `product_id`                 | string  | Identificador do produto   | CSV de origem          |
| `product_category_name`      | string  | Categoria do produto       | CSV de origem          |
| `product_name_lenght`        | integer | Comprimento do nome        | CSV de origem          |
| `product_description_lenght` | integer | Comprimento da descrição   | CSV de origem          |
| `product_photos_qty`         | integer | Quantidade de fotos        | CSV de origem          |
| `product_weight_g`           | integer | Peso em gramas             | CSV de origem          |
| `product_length_cm`          | integer | Comprimento em centímetros | CSV de origem          |
| `product_height_cm`          | integer | Altura em centímetros      | CSV de origem          |
| `product_width_cm`           | integer | Largura em centímetros     | CSV de origem          |


#### bronze_sellers

Finalidade: armazenar os dados cadastrais e de localização dos vendedores.

| Campo                    | Tipo    | Domínio / significado     | Origem / transformação |
| ------------------------ | ------- | ------------------------- | ---------------------- |
| `seller_id`              | string  | Identificador do vendedor | CSV de origem          |
| `seller_zip_code_prefix` | integer | Prefixo do CEP            | CSV de origem          |
| `seller_city`            | string  | Município do vendedor     | CSV de origem          |
| `seller_state`           | string  | UF do vendedor            | CSV de origem          |


#### bronze_category_translation

Finalidade: armazenar a correspondência entre nomes de categorias de produtos e suas versões em inglês.

| Campo                           | Tipo   | Domínio / significado       | Origem / transformação |
| ------------------------------- | ------ | --------------------------- | ---------------------- |
| `product_category_name`         | string | Nome da categoria na origem | CSV de origem          |
| `product_category_name_english` | string | Nome traduzido da categoria | CSV de origem          |

2. Camada Silver

Na Silver, as tabelas são carregadas da Bronze e submetidas a procedimentos comuns de tratamento:

- remoção de espaços desnecessários;
- transformação de strings vazias em NULL;
- padronização de caixa em campos selecionados;
- conversão de tipos;
- remoção de duplicidades;
- remoção de registros sem chaves definidas;
- criação de variáveis derivadas na tabela de pedidos.
- silver_customers

Finalidade: disponibilizar os dados de clientes após padronização e deduplicação.

Transformações efetivamente realizadas:

- limpeza dos campos customer_id, customer_unique_id, customer_city e customer_state;
- customer_state convertido para maiúsculas;
- customer_zip_code_prefix convertido para integer;
- remoção de duplicidades;
- remoção de registros sem customer_id.

Campos: os mesmos cinco campos da bronze_customers, com os tipos tratados acima.

#### silver_geolocation

Finalidade: disponibilizar os dados geográficos tratados.

Transformações efetivamente realizadas:

- limpeza de geolocation_city e geolocation_state;
- geolocation_state convertido para maiúsculas;
- conversão de CEP para integer;
- conversão de latitude e longitude para double;
- remoção de duplicidades exatas.

Campos: os mesmos cinco campos da Bronze.

#### silver_order_items

Finalidade: disponibilizar os itens de pedidos com identificadores e valores padronizados.

Transformações efetivamente realizadas:

- limpeza de order_id, product_id e seller_id;
- conversão de order_item_id para integer;
- conversão de price e freight_value para double;
- remoção de duplicidades;
- remoção de registros sem order_id ou order_item_id.

Campos: os mesmos sete campos da Bronze.

#### silver_order_payments

Finalidade: disponibilizar os pagamentos tratados.

Transformações efetivamente realizadas:

- limpeza de order_id e payment_type;
- payment_type convertido para minúsculas;
- conversão de payment_sequential e payment_installments para integer;
- conversão de payment_value para double;
- remoção de duplicidades;
- remoção de registros sem order_id ou payment_sequential.

Campos: os mesmos cinco campos da Bronze.

#### silver_order_reviews

Finalidade: disponibilizar as avaliações em formato adequado para análise quantitativa.

| Campo                     | Tipo      | Tratamento realizado                                                      |
| ------------------------- | --------- | ------------------------------------------------------------------------- |
| `review_id`               | string    | Limpeza e deduplicação                                                    |
| `order_id`                | string    | Limpeza e deduplicação                                                    |
| `review_score`            | integer   | Conversão somente de valores de 1 a 5; valores inválidos tornam-se `NULL` |
| `review_comment_title`    | string    | Limpeza                                                                   |
| `review_comment_message`  | string    | Limpeza                                                                   |
| `review_creation_date`    | timestamp | Conversão com `try_to_timestamp`                                          |
| `review_answer_timestamp` | timestamp | Conversão com `try_to_timestamp`                                          |
A regra de validação considera como domínio válido da avaliação os valores inteiros de 1 a 5.


#### silver_orders

Finalidade: disponibilizar os pedidos tratados e acrescentar indicadores necessários à análise de entrega.

| Campo                           | Tipo      | Origem / transformação                                      |
| ------------------------------- | --------- | ----------------------------------------------------------- |
| `order_id`                      | string    | Bronze + limpeza/deduplicação                               |
| `customer_id`                   | string    | Bronze + limpeza                                            |
| `order_status`                  | string    | Bronze + limpeza e conversão para minúsculas                |
| `order_purchase_timestamp`      | timestamp | Bronze + tipagem                                            |
| `order_approved_at`             | timestamp | Bronze + tipagem                                            |
| `order_delivered_carrier_date`  | timestamp | Bronze + tipagem                                            |
| `order_delivered_customer_date` | timestamp | Bronze + tipagem                                            |
| `order_estimated_delivery_date` | date      | Bronze + conversão para date                                |
| `order_purchase_date`           | date      | Derivada de `order_purchase_timestamp`                      |
| `delivery_days`                 | integer   | Diferença entre data de entrega ao cliente e data da compra |
| `delivery_delay_days`           | integer   | Diferença entre entrega real e data estimada                |
| `delivery_status`               | string    | Classificação derivada                                      |

O domínio de delivery_status é definido diretamente pelo código:
`NAO_ENTREGUE`
`ENTREGUE_COM_ATRASO`
`ENTREGUE_NO_PRAZO`

Para `delivery_delay_days`, valores positivos representam atraso e valores negativos representam entrega antecipada.

#### silver_products

Finalidade: disponibilizar os produtos tratados.

Transformações efetivamente realizadas:

- limpeza de product_id e product_category_name;
- categoria convertida para minúsculas;
- conversão dos campos quantitativos para integer;
- remoção de duplicidades;
- remoção de registros sem product_id.

Os nove campos são mantidos.

#### silver_sellers

Finalidade: disponibilizar os vendedores tratados.

Transformações efetivamente realizadas:

- limpeza de seller_id, seller_city e seller_state;
- seller_state convertido para maiúsculas;
- CEP convertido para integer;
- remoção de duplicidades;
- remoção de registros sem seller_id.

silver_category_translation

Finalidade: disponibilizar a tabela de tradução de categorias tratada.

Transformações efetivamente realizadas:

- limpeza dos dois campos;
- ambos convertidos para minúsculas;
- remoção de duplicidades;
- remoção de registros sem product_category_name.

### 3.1 Camada Gold

#### gold_dim_customer

Finalidade: dimensão de clientes utilizada para enriquecer as tabelas fato.
| Campo                      | Tipo    | Origem             |
| -------------------------- | ------- | ------------------ |
| `customer_id`              | string  | `silver_customers` |
| `customer_unique_id`       | string  | `silver_customers` |
| `customer_zip_code_prefix` | integer | `silver_customers` |
| `customer_city`            | string  | `silver_customers` |
| `customer_state`           | string  | `silver_customers` |

#### gold_dim_product

Finalidade: dimensão de produtos com a tradução das categorias.
| Campo                           | Tipo    | Origem                        |
| ------------------------------- | ------- | ----------------------------- |
| `product_id`                    | string  | `silver_products`             |
| `product_category_name`         | string  | `silver_products`             |
| `product_category_name_english` | string  | `silver_category_translation` |
| `product_name_lenght`           | integer | `silver_products`             |
| `product_description_lenght`    | integer | `silver_products`             |
| `product_photos_qty`            | integer | `silver_products`             |
| `product_weight_g`              | integer | `silver_products`             |
| `product_length_cm`             | integer | `silver_products`             |
| `product_height_cm`             | integer | `silver_products`             |
| `product_width_cm`              | integer | `silver_products`             |


#### gold_dim_seller

Finalidade: dimensão de vendedores utilizada no fato de itens de venda.
| Campo                    | Tipo    | Origem           |
| ------------------------ | ------- | ---------------- |
| `seller_id`              | string  | `silver_sellers` |
| `seller_zip_code_prefix` | integer | `silver_sellers` |
| `seller_city`            | string  | `silver_sellers` |
| `seller_state`           | string  | `silver_sellers` |


#### gold_dim_date

Finalidade: dimensão temporal derivada das datas de compra dos pedidos.

| Campo        | Tipo    | Origem / transformação         |
| ------------ | ------- | ------------------------------ |
| `date`       | date    | `order_purchase_timestamp`     |
| `year`       | integer | Ano derivado de `date`         |
| `month`      | integer | Mês derivado de `date`         |
| `month_name` | string  | Nome do mês derivado de `date` |
| `quarter`    | integer | Trimestre derivado de `date`   |
| `year_month` | string  | Ano-mês derivado de `date`     |


### Tabelas fato

#### gold_fact_sales_items

Granularidade: um registro por item de pedido.

| Campo                      | Tipo    | Origem / transformação                               |
| -------------------------- | ------- | ---------------------------------------------------- |
| `order_id`                 | string  | `silver_order_items`                                 |
| `order_item_id`            | integer | `silver_order_items`                                 |
| `product_id`               | string  | `silver_order_items`                                 |
| `seller_id`                | string  | `silver_order_items`                                 |
| `customer_id`              | string  | `silver_orders`                                      |
| `order_date`               | date    | Derivada de `silver_orders.order_purchase_timestamp` |
| `product_category`         | string  | `gold_dim_product`                                   |
| `product_category_english` | string  | `gold_dim_product`                                   |
| `customer_state`           | string  | `gold_dim_customer`                                  |
| `seller_state`             | string  | `gold_dim_seller`                                    |
| `item_price`               | double  | `silver_order_items.price`                           |
| `freight_value`            | double  | `silver_order_items.freight_value`                   |
| `item_revenue`             | double  | Igual a `item_price`                                 |

#### gold_fact_orders

Granularidade: um registro por pedido.

| Campo                 | Tipo    | Origem / transformação                          |
| --------------------- | ------- | ----------------------------------------------- |
| `order_id`            | string  | `silver_orders`                                 |
| `customer_id`         | string  | `silver_orders`                                 |
| `order_date`          | date    | Derivada da data/hora da compra                 |
| `customer_state`      | string  | `gold_dim_customer`                             |
| `order_status`        | string  | `silver_orders`                                 |
| `order_item_value`    | double  | Soma de `price` por pedido                      |
| `order_freight_value` | double  | Soma de `freight_value` por pedido              |
| `item_count`          | bigint  | Contagem distinta de `order_item_id` por pedido |
| `payment_value`       | double  | Soma de `payment_value` por pedido              |
| `delivery_days`       | integer | `silver_orders`                                 |
| `delivery_delay_days` | integer | `silver_orders`                                 |
| `delivery_status`     | string  | `silver_orders`                                 |


#### gold_fact_delivery_reviews

Granularidade: um registro por pedido que possui avaliação.

| Campo                  | Tipo    | Origem / transformação             |
| ---------------------- | ------- | ---------------------------------- |
| `order_id`             | string  | `silver_orders`                    |
| `order_status`         | string  | `silver_orders`                    |
| `delivery_status`      | string  | `silver_orders`                    |
| `delivery_days`        | integer | `silver_orders`                    |
| `delivery_delay_days`  | integer | `silver_orders`                    |
| `average_review_score` | double  | Média de `review_score` por pedido |
| `review_count`         | bigint  | Contagem de avaliações por pedido  |

### Tabelas Gold das perguntas de negócio

#### gold_q1_category_revenue

Finalidade: responder à Q1 — categorias com maior faturamento.

| Campo                      | Tipo   | Origem / transformação                                                            |
| -------------------------- | ------ | --------------------------------------------------------------------------------- |
| `product_category`         | string | Categoria de `fact_sales_items`; nulos substituídos por `Sem categoria informada` |
| `product_category_english` | string | Categoria traduzida; nulos substituídos por `Uncategorized`                       |
| `revenue`                  | double | Soma de `item_revenue` por categoria                                              |
| `revenue_share_pct`        | double | Faturamento da categoria / faturamento total × 100                                |


#### gold_q2_state_sales

Finalidade: responder à Q2 — volume de pedidos e faturamento por estado.

| Campo            | Tipo   | Origem / transformação                  |
| ---------------- | ------ | --------------------------------------- |
| `customer_state` | string | Estado do cliente em `fact_orders`      |
| `order_count`    | bigint | Contagem distinta de pedidos por estado |
| `revenue`        | double | Soma de `order_item_value` por estado   |


#### gold_q3_delivery_review

Finalidade: responder à Q3 — relação entre situação da entrega e avaliação.

| Campo                  | Tipo   | Origem / transformação                                    |
| ---------------------- | ------ | --------------------------------------------------------- |
| `delivery_status`      | string | Mantém apenas `ENTREGUE_COM_ATRASO` e `ENTREGUE_NO_PRAZO` |
| `order_count`          | bigint | Contagem distinta de pedidos                              |
| `average_review_score` | double | Média de `average_review_score`                           |


#### gold_q4_freight

Finalidade: responder à Q4 — frete médio por estado e categoria.

| Campo                      | Tipo   | Origem / transformação       |
| -------------------------- | ------ | ---------------------------- |
| `customer_state`           | string | Estado do cliente            |
| `product_category`         | string | Categoria do produto         |
| `product_category_english` | string | Categoria traduzida          |
| `average_freight`          | double | Média de `freight_value`     |
| `order_count`              | bigint | Contagem distinta de pedidos |


#### gold_q5_temporal

Finalidade: responder à Q5 — evolução temporal de pedidos, faturamento e ticket médio.

| Campo            | Tipo   | Origem / transformação          |
| ---------------- | ------ | ------------------------------- |
| `order_date`     | date   | Data do pedido em `fact_orders` |
| `order_count`    | bigint | Contagem distinta de pedidos    |
| `revenue`        | double | Soma de `order_item_value`      |
| `average_ticket` | double | `revenue / order_count`         |




------------------------------------------------------------------------

## 4. Pipeline de Dados

O processo foi dividido em cinco notebooks:

- [01_Bronze](./notebooks/01_bronze.ipynb)
- [02_Silver](./notebooks/02_Silver.ipynb)
- [03_Gold](./notebooks/03_Gold.ipynb)
- [04_Qualidade_Dados](./notebooks/04_Qualidade_Dados.ipynb)
- [05_Analise](./notebooks/05_Analise.ipynb)

Essa divisão separa as etapas de ingestão, transformação, modelagem,
validação da qualidade e análise.

### 4.1 Bronze --- ingestão

O Notebook 01 realiza a leitura dos arquivos de origem e sua
persistência na camada Bronze.

### 4.2 Silver --- tratamento

O Notebook 02 utiliza as tabelas Bronze e realiza os tratamentos
necessários para produzir dados padronizados e preparados para
modelagem.

### 4.3 Gold --- modelagem analítica

O Notebook 03 utiliza a camada Silver para construir dimensões, fatos e
tabelas analíticas.

### 4.4 Qualidade

O Notebook 04 verifica a qualidade dos dados tratados, considerando
completude, consistência, unicidade, validade/acurácia e outliers.

### 4.5 Análise

O Notebook 05 utiliza as tabelas Gold para responder às cinco perguntas
de negócio por meio de tabelas, consultas e visualizações.

### Evidência 4 --- listagem das tabelas no databricks

<img width="1585" height="844" alt="image" src="https://github.com/user-attachments/assets/f764d6de-cc52-4ce9-b002-ca47d9c25b5e" />

<img width="1570" height="758" alt="image" src="https://github.com/user-attachments/assets/8ab0554e-cee2-4cd3-a2c6-e1599ad83819" />

------------------------------------------------------------------------

## 5. Qualidade de Dados

A qualidade dos dados foi analisada antes da etapa final de análise,
considerando cinco dimensões:

-   completude;
-   consistência;
-   unicidade;
-   validade/acurácia;
-   outliers.

A análise foi realizada no Notebook 04.

### 5.1 Completude

Foram verificadas as principais tabelas quanto à presença de valores
nulos e ausentes.

Os resultados foram utilizados para identificar campos que exigiam
tratamento e avaliar a possibilidade de utilização dessas informações
nas análises.

### 5.2 Consistência

Foram verificadas regras relacionadas aos tipos e valores esperados dos
campos, incluindo:

-   consistência temporal;
-   valores de `review_score`;
-   status de pedidos;
-   indicadores de entrega;
-   valores monetários;
-   categorias;
-   campos derivados.

### 5.3 Unicidade

A unicidade foi avaliada considerando as chaves relevantes de cada
tabela.

Na tabela de avaliações, a deduplicação da camada Silver considerou
`review_id + order_id`.

Na etapa de qualidade, também foram investigadas ocorrências de
`review_id` repetidos. Essa análise mostrou que a repetição isolada do
identificador não deve ser interpretada automaticamente como duplicidade
de registro, uma vez que a avaliação foi analisada em conjunto com o
pedido.

### 5.4 Validade e acurácia

Foram verificadas regras de domínio, incluindo a escala das avaliações.

No tratamento realizado na Silver, valores inválidos de `review_score`
foram convertidos para `NULL`, enquanto as notas válidas foram mantidas
na escala de 1 a 5.

Também foram verificadas regras relacionadas aos indicadores de entrega
e aos valores monetários.

### 5.5 Outliers

Foram avaliados valores extremos em variáveis numéricas, especialmente
preço e frete.

A identificação estatística de um outlier não foi utilizada
automaticamente como critério de exclusão. Valores extremos podem
representar observações reais da operação e, por isso, foram analisados
considerando seu contexto antes de qualquer decisão de tratamento.

### Evidência 5 --- qualidade (print seguido da reprodução do markdown)

<img width="1248" height="747" alt="image" src="https://github.com/user-attachments/assets/4a0e230d-3fd2-4636-809b-03f63838aab6" />


### Principais problemas identificados e tratamentos realizados

A análise de qualidade mostrou que a base apresenta alguns problemas de completude, duplicidade e padronização, mas não foram identificadas inconsistências nas principais regras de negócio verificadas. Os tratamentos realizados na camada Silver buscaram corrigir problemas estruturais sem eliminar informações que poderiam representar situações reais da operação.

#### Avaliações dos clientes

A tabela de avaliações apresentou **104.162 registros na Bronze e 101.885 na Silver**. Durante o tratamento, foram removidas **104 duplicidades exatas** e **2.173 registros sem algum dos identificadores essenciais (`review_id` ou `order_id`)**.

Também foram identificados **2.558 valores de `review_score` que não seguiam o padrão esperado de 1 a 5**. Esses valores foram convertidos para `NULL` na Silver, em vez de receberem uma classificação arbitrária. Após o tratamento, não permaneceram avaliações fora do intervalo de 1 a 5.

Os campos textuais de comentário apresentam proporção elevada de valores nulos, especialmente `review_comment_title` (88,23%) e `review_comment_message` (59,69%). Esses valores foram preservados, pois a ausência de comentário representa uma situação possível na interação do cliente com a plataforma e não havia informação suficiente para preenchê-los de forma confiável.

#### Geolocalização

A tabela de geolocalização apresentou **1.000.163 registros na Bronze e 738.332 na Silver**. A redução ocorreu pela remoção de **261.831 duplicidades exatas**, mantendo uma única ocorrência de cada registro efetivamente repetido.

#### Pedidos e informações de entrega

A tabela de pedidos manteve **99.441 registros** entre Bronze e Silver. Foram identificados **2.965 pedidos sem data efetiva de entrega**. Esses valores foram preservados, pois a ausência da data é compatível com pedidos que não foram entregues e não deve ser interpretada automaticamente como erro de preenchimento.

A camada Silver também passou a contar com os indicadores `delivery_days`, `delivery_delay_days` e `delivery_status`, que permitem classificar a situação das entregas e apoiar as análises posteriores.

Valores negativos de `delivery_delay_days` não foram tratados como erro: eles representam entregas realizadas antes da data estimada. A classificação utilizada considera como atraso somente valores positivos.

#### Produtos e demais tabelas

Alguns atributos de produtos apresentam valores nulos, principalmente os relacionados às características descritivas do produto. Esses valores foram mantidos quando não havia informação confiável para preenchimento.

Nas demais tabelas analisadas, as verificações de chaves, tipos e valores permitiram confirmar a consistência estrutural dos dados utilizados no pipeline.

#### Valores extremos

A análise pelo método do intervalo interquartil (IQR) identificou valores extremos em `price` e `freight_value`. Foram encontrados **8.427 outliers de preço (7,48%)** e **12.134 outliers de frete (10,77%)**.

Esses registros não foram removidos automaticamente. A identificação estatística de um valor como outlier não significa, por si só, que ele seja um erro. Como preços e fretes podem variar legitimamente entre produtos, pedidos e localidades, os valores foram preservados para não excluir observações potencialmente válidas das análises.

#### Síntese dos tratamentos

De forma geral, a camada Silver concentrou os tratamentos necessários para:

- padronizar campos textuais e tipos de dados;
- remover duplicidades exatas;
- remover registros sem identificadores essenciais quando necessário;
- tratar valores inválidos de `review_score`;
- converter datas para tipos apropriados;
- criar indicadores derivados relacionados à entrega;
- preservar valores nulos quando eles possuem significado no contexto da operação.

------------------------------------------------------------------------

## 6. Análise de Dados

As análises foram realizadas no Notebook 05 utilizando as tabelas
analíticas construídas na camada Gold.

### 6.1 Q1 --- Quais categorias concentram o maior faturamento?

O faturamento apresenta uma concentração relevante nas categorias de
maior desempenho. **beleza_saude**, **relogios_presentes** e
**cama_mesa_banho** ocupam as três primeiras posições, com participações
de **9,26%**, **8,87%** e **7,63%** do faturamento, respectivamente.

As dez categorias com maior faturamento representam, em conjunto,
**62,36% da receita analisada**, mostrando que uma parcela significativa
do faturamento está concentrada em um grupo relativamente reduzido de
categorias. Ao mesmo tempo, a presença de diferentes segmentos entre as
primeiras posições indica que essa concentração não está associada a um
único tipo de produto.

Para o contexto do negócio, esse resultado ajuda a identificar as
categorias com maior peso na geração de receita e fornece uma referência
para o acompanhamento do desempenho comercial. A visualização das 20
primeiras categorias amplia essa leitura e permite observar como o
faturamento se distribui após o grupo de maior participação.

### Evidência 6 --- Q1

<img width="1089" height="619" alt="image" src="https://github.com/user-attachments/assets/74ee41ee-becb-4377-bac1-de246cda4885" />


<img width="1078" height="621" alt="image" src="https://github.com/user-attachments/assets/1e0acb51-4283-4a54-9901-d6d9c2f3d794" />


------------------------------------------------------------------------

### 6.2 Q2 --- Quais estados concentram maior volume e faturamento?

A distribuição dos pedidos apresenta uma forte concentração nos estados
com maior volume de vendas. **São Paulo lidera tanto em quantidade de
pedidos quanto em faturamento**, com **41.746 pedidos** e
aproximadamente **R\$ 5,20 milhões** em receita. Na sequência aparecem
**Rio de Janeiro**, com **12.852 pedidos** e **R\$ 1,82 milhão**, e
**Minas Gerais**, com **11.635 pedidos** e **R\$ 1,59 milhão**.

O resultado mostra que os estados com maior volume de pedidos também
ocupam as primeiras posições em faturamento, embora em proporções
diferentes. Isso indica que a dimensão territorial tem peso relevante na
distribuição da atividade comercial observada na base.

Para o contexto do negócio, essa concentração ajuda a identificar os
principais mercados em termos de demanda e geração de receita. Ao mesmo
tempo, é relevante ter em mente que a análise deve ser entendida como um
retrato do período abrangido pela base, não como uma representação atual
da distribuição do comércio eletrônico brasileiro.

### Evidência 7 --- Q2

<img width="1095" height="521" alt="image" src="https://github.com/user-attachments/assets/e946adc1-ba2e-4430-8af6-146731af84a9" />


<img width="1128" height="574" alt="image" src="https://github.com/user-attachments/assets/17d376a2-2465-47e0-b102-70661f5cb89c" />


------------------------------------------------------------------------

### 6.3 Q3 --- Pedidos atrasados apresentam avaliações menores?

A análise compara diretamente a avaliação média dos pedidos
classificados como entregues no prazo e dos pedidos entregues com
atraso.

Os dados indicam uma diferença expressiva entre os dois grupos. A
avaliação média foi de **4,29** entre os pedidos entregues no prazo,
enquanto os pedidos entregues com atraso apresentaram média de **2,27**,
uma diferença de **2,02 pontos**.

Esse resultado sugere uma associação relevante entre o desempenho da
entrega e a percepção do cliente sobre a experiência de compra. Na base
analisada, os pedidos que não cumpriram o prazo estimado estão
associados a avaliações consideravelmente menores.

O resultado é relevante para o negócio porque evidencia a dimensão da
logística na experiência do cliente. Entretanto, a análise não permite
afirmar que o atraso, isoladamente, seja a causa das avaliações menores,
uma vez que outros fatores da experiência de compra também podem
influenciar a satisfação. Assim, o resultado deve ser interpretado como
uma associação observada nos dados e como um possível ponto de atenção
para a gestão da operação logística.

### Evidência 8 --- Q3

<img width="1086" height="543" alt="image" src="https://github.com/user-attachments/assets/f86851fd-1a8e-407c-b7ca-f11b811706b4" />


<img width="1102" height="387" alt="image" src="https://github.com/user-attachments/assets/b41db99a-7e7f-41c2-af9c-d3597d763f74" />


------------------------------------------------------------------------

### 6.4 Q4 --- Como o frete varia por estado e categoria?

O frete apresenta diferenças relevantes tanto entre os estados quanto
entre as categorias de produtos.

Na comparação por estado, **RR apresenta o maior frete médio, de R\$
42,54**, enquanto **SP apresenta o menor, de R\$ 15,08**. Essa diferença
deve ser interpretada junto ao volume de pedidos: RR possui apenas **46
pedidos** na base analisada, enquanto SP concentra **41.706**, o que
torna o resultado de SP mais representativo em termos de volume.

Por categoria, **pcs apresenta o maior frete médio (R\$ 49,11)**,
seguida por **eletrodomesticos_2 (R\$ 44,60)** e por algumas categorias
de móveis, com valores de **R\$ 42,91**. Entretanto, parte dessas
categorias possui baixo volume de pedidos.

Entre categorias com maior quantidade de pedidos, `moveis_escritorio`,
por exemplo, apresenta frete médio de **R\$ 40,50 em 1.273 pedidos**,
enquanto `ferramentas_jardim` apresenta **R\$ 22,78 em 3.518 pedidos**.

A análise das combinações entre estado e categoria reforça que o frete
não depende apenas do tipo de produto, mas também do destino. A
categoria `cama_mesa_banho`, por exemplo, apresenta frete médio de **R\$
14,90 em SP, com 4.416 pedidos**, e **R\$ 19,56 no RJ, com 1.393
pedidos**.

Dessa forma, os resultados indicam que as diferenças logísticas estão
associadas à combinação entre características da categoria e localização
do cliente, sendo importante considerar também o volume de pedidos ao
interpretar valores extremos.

### Evidência 9 --- Q4

<img width="1129" height="627" alt="image" src="https://github.com/user-attachments/assets/9a7ebf12-1354-44ef-af80-9e10388623c5" />


<img width="1110" height="561" alt="image" src="https://github.com/user-attachments/assets/92cb14c0-a3ba-437b-a8c4-cb825381d73c" />


<img width="1130" height="544" alt="image" src="https://github.com/user-attachments/assets/1bcd6e51-f623-4bab-b7cd-7868a8de2f16" />


------------------------------------------------------------------------

### 6.5 Q5 --- Como pedidos, faturamento e ticket médio evoluíram ao longo do tempo?

A evolução mensal mostra uma expansão expressiva da operação ao longo do
período analisado.

O volume passou de **789 pedidos em janeiro de 2017** para mais de **7
mil pedidos em janeiro e março de 2018**, enquanto o maior volume da
série ocorreu em **novembro de 2017, com 7.451 pedidos**. O faturamento
acompanha essa expansão, alcançando aproximadamente **R\$ 1,01 milhão no
mesmo mês**.

O crescimento do faturamento está associado principalmente ao aumento do
volume de pedidos, e não a uma elevação contínua do ticket médio. Entre
os meses com maior volume de pedidos, o ticket médio oscila
aproximadamente entre **R\$ 125 e R\$ 145**, sem apresentar uma
trajetória contínua de crescimento. Em abril e maio de 2018, por
exemplo, o ticket médio volta a alcançar cerca de **R\$ 144--145**,
mesmo com o volume permanecendo elevado.

Assim, os resultados indicam uma operação que ganhou escala ao longo do
período, com o aumento da quantidade de pedidos exercendo papel central
na expansão do faturamento. Ao mesmo tempo, as oscilações do ticket
mostram que o crescimento da receita não dependeu de aumentos sucessivos
no valor médio das compras.

Os registros de setembro e dezembro de 2016 e setembro de 2018
apresentam volumes muito baixos e, por isso, têm pouca
representatividade para caracterizar a tendência da operação. A
interpretação da evolução temporal deve se concentrar principalmente no
período em que a base apresenta maior volume de pedidos.

### Evidência 10 --- Q5

<img width="1108" height="580" alt="image" src="https://github.com/user-attachments/assets/ee233da6-c03e-4087-887f-32d7c07b3b41" />


<img width="1091" height="543" alt="image" src="https://github.com/user-attachments/assets/820e7240-03fc-4f06-b899-31234a824a94" />


<img width="1047" height="526" alt="image" src="https://github.com/user-attachments/assets/4822b6f7-9219-47c5-a006-3a571c5a80e9" />


------------------------------------------------------------------------

### 6.6 Discussão geral dos resultados

As cinco análises apresentam perspectivas diferentes sobre a mesma
operação e, quando observadas em conjunto, ajudam a construir uma visão
mais completa do e-commerce analisado.

Do ponto de vista comercial, o faturamento apresenta concentração entre
algumas categorias, com **beleza_saude**, **relogios_presentes** e
**cama_mesa_banho** nas primeiras posições. No conjunto, as dez
categorias de maior faturamento representam **62,36% da receita
analisada**. Essa concentração, entretanto, não significa dependência de
uma única categoria: diferentes segmentos aparecem entre os maiores
geradores de receita. Territorialmente, também existe uma concentração
importante, com **São Paulo** apresentando o maior volume de pedidos e o
maior faturamento entre os estados analisados.

A dimensão logística acrescenta uma perspectiva importante a essa
leitura. Os pedidos entregues com atraso apresentam avaliação média de
**2,27**, contra **4,29** entre aqueles entregues no prazo. A diferença
de **2,02 pontos** evidencia uma associação relevante entre o desempenho
da entrega e a avaliação dos clientes, embora os dados não permitam
atribuir causalidade ao atraso isoladamente. O frete, por sua vez,
apresenta diferenças expressivas entre estados e categorias, reforçando
que a dimensão logística não é uniforme dentro da operação e precisa ser
analisada considerando o contexto de cada pedido.

A análise temporal mostra outro aspecto importante: o crescimento
observado no período está associado principalmente à expansão do volume
de pedidos. O faturamento acompanha esse movimento, enquanto o ticket
médio apresenta oscilações, mas não uma trajetória contínua de
crescimento. Isso ajuda a diferenciar um aumento de receita decorrente
de maior escala de um crescimento provocado pelo aumento do valor médio
das compras.

Considerando os resultados em conjunto, o MVP mostra que o desempenho de
uma operação de e-commerce precisa ser observado por mais de uma
perspectiva. A receita mostra onde estão os principais segmentos
comerciais; a distribuição territorial indica onde está concentrada a
demanda; os indicadores de entrega ajudam a relacionar a operação
logística à experiência do cliente; o frete evidencia diferenças entre
contextos operacionais; e a análise temporal mostra como esses
indicadores se comportam ao longo do período.

Essa integração também evidencia o papel das etapas anteriores do
pipeline. As informações necessárias para responder às perguntas estavam
distribuídas entre diferentes tabelas da base original. O tratamento, a
integração e a organização desses dados nas camadas Silver e Gold
permitiram transformar fontes operacionais em estruturas que poderiam
ser consultadas de maneira direcionada às perguntas de negócio. Dessa
forma, a análise apresentada aqui é o resultado de todo o fluxo
construído no MVP, e não apenas de consultas realizadas sobre os
arquivos originais.

Por fim, os resultados precisam ser interpretados dentro dos limites da
base. Os dados são históricos e abrangem um período específico, de modo
que os padrões identificados descrevem o comportamento observado nesse
conjunto e não devem ser tratados como uma representação atual do
comércio eletrônico brasileiro. Para uma aplicação empresarial real,
seria necessário complementar essa análise com dados mais recentes,
informações adicionais sobre custos e operação e acompanhamento contínuo
dos indicadores.

------------------------------------------------------------------------

## 7. Considerações Finais e Autoavaliação

A construção deste MVP permitiu acompanhar, na prática, todo o caminho
entre uma pergunta de negócio e uma análise baseada em dados. O projeto
não se limitou à execução de consultas: foi necessário organizar
diferentes fontes, compreender seus relacionamentos, tratar
inconsistências, definir estruturas analíticas e, somente depois,
utilizar os dados para responder às perguntas propostas.

Um dos principais aprendizados foi perceber o quanto as decisões tomadas
nas etapas de preparação condicionam a qualidade da análise final. A
definição das chaves, a padronização dos campos, o tratamento dos dados
e, principalmente, a escolha da granularidade das tabelas Gold tiveram
impacto direto sobre o que poderia ser analisado posteriormente. A
própria construção da Q5, por exemplo, mostrou a importância de adequar
a granularidade à pergunta: para avaliar a evolução da operação, os
dados precisaram ser agregados mensalmente, em vez de simplesmente
apresentados no nível diário.

Outro aprendizado importante foi entender que responder a uma pergunta
de negócio exige mais do que encontrar um valor em uma tabela. Na
análise da relação entre entrega e avaliação, por exemplo, os resultados
mostraram uma diferença expressiva entre os grupos, mas também foi
necessário reconhecer que essa associação não é suficiente para
estabelecer uma relação de causa e efeito. Da mesma forma, na análise do
frete, valores extremos precisaram ser observados junto ao volume de
pedidos para evitar interpretações baseadas apenas em médias de grupos
muito pequenos.

Nesse sentido, considero que o MVP atingiu seu objetivo principal:
construir um fluxo de dados capaz de transformar informações
originalmente distribuídas em diferentes fontes em respostas
estruturadas para perguntas concretas de negócio. O resultado também
mostrou que a etapa de análise depende diretamente da qualidade das
etapas anteriores e que uma solução de Engenharia de Dados precisa ser
pensada considerando não apenas a disponibilização dos dados, mas também
seu uso posterior.

Como oportunidade de melhoria futura, vejo que o projeto poderia
incorporar dados mais recentes e novas dimensões de análise, além de
mecanismos de atualização automatizada e uma camada de visualização
voltada ao acompanhamento contínuo dos indicadores. Também seria
possível aprofundar algumas das relações identificadas neste MVP,
especialmente a relação entre desempenho logístico e satisfação dos
clientes e as diferenças de frete entre diferentes combinações de
território e categoria.

De forma geral, o principal aprendizado deste projeto foi compreender
melhor a passagem de um problema de negócio para uma solução de dados.

------------------------------------------------------------------------

## 8. Estrutura do Repositório

A entrega é composta pelos notebooks utilizados nas diferentes etapas do
pipeline e por este README, que documenta o contexto, a arquitetura, a
qualidade dos dados e os resultados analíticos.

------------------------------------------------------------------------

## 9. Tecnologias Utilizadas

-   **Databricks** --- ambiente de desenvolvimento e execução do
    pipeline;
-   **Apache Spark / PySpark** --- processamento e transformação dos
    dados;
-   **SQL** --- consultas e análises;
-   **Python** --- desenvolvimento das transformações e análises;
-   **Delta Lake** --- persistência das tabelas;
-   **GitHub** --- versionamento e disponibilização do código.

------------------------------------------------------------------------

## 10. Referências

-   Olist --- Brazilian E-Commerce Public Dataset\
    https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

-   Databricks Documentation\
    https://docs.databricks.com/
