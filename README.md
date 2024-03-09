# Análise de dados no Power BI do dataset de e-commerce da Olist, a partir de ETL no Databricks

## Apresentação:
Este projeto realiza o estudo de uma base de dados pública da Olist Store, amplamente conhecida plataforma de soluções de e-commerce, a partir do desenvolvimento de todas as etapas de um fluxo de análise de dados, desde a extração do dado de origem até a sua visualização final.

A base de dados é disponibilizada pela plataforma Kaggle. A etapa do ETL é realizada no Databricks, com a estrutura de camadas bronze, silver e gold, e utilizando as linguagens de programação Python e SQL. E o painel com a visualização dos dados é apresentado no Power BI. A partir da imagem a seguir, é possível visualizar o esquema da arquitetura do fluxo do dado.

![Arquitetura do projeto](image/arquitetura.png)


## Como executar este projeto:
1. Possuir conta ativa no Databricks
2. Acessar plataforma e importar a base de dados de origem (arquivos .csv)
3. Importar os notebooks (arquivos .dbc), associá-los a um cluster ativo e executá-los de forma automática clicando em Run All, seguindo a sequência de Camada Bronze > Camada Silver > Camada Gold, executando um notebook de cada vez
4. Importar as tabelas da Camada Gold no Power BI e criar seu próprio dashboard ou simplesmente abrir o arquivo .pbix e visualizar o dashboard aqui apresentado


## Informações sobre a base de dados:
Nossa base de dados foi obtida por meio da plataforma Kaggle, que distribui de forma livre e gratuita bases de dados para estudo, projetos e competições internas, e é composta por dados reais do e-commerce da plataforma Olist. Essa base tem informações de cerca de 100 mil pedidos realizados entre os anos de 2016 e 2018, de diversos marketplaces no Brasil, com dados de clientes, vendedores, produtos, avaliações de pedidos e mais. Para obter informações mais detalhadas e acessar a base publicada, segue link: <https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce/>

Para uma melhor compreensão inicial da base de dados, segue uma breve descrição do que contém em cada arquivo de origem:
* **olist_customer_dataset** - lista os clientes e a localização de cada. O campo customer_id é um id único criado para cada customer em cada compra, logo o mesmo customer poderá ter mais de um customer_id associado, caso tenha realizado mais de uma compra. Já o customer_unique_id é o id que referencia cada cliente unicamente.
* **olist_geolocation_dataset** - localização geográfica, prefixo de cep, cidade e estado.
* **olist_order_items_dataset** - lista os produtos de cada pedido, o vendedor responsável, valores e a data limite de entrega.
* **olist_order_payments_dataset** - tipo do pagamento utilizado em cada compra, número de parcelas e valor.
* **olist_order_reviews_dataset** - nota de avaliação e comentário do cliente para a compra realizada, data de criação e data de resposta.
* **olist_orders_dataset** - lista os pedidos realizados, com indicação do cliente, status do pedido e a data/hora da compra, da confirmação do pagamento, do envio do pedido, da entrega ao cliente e da estimativa de entrega.
* **olist_products_dataset** - lista os produtos vendidos, nome, descrição, quantidade de fotos, peso, dimensões e nome da categoria de cada produto.
* **olist_sellers_dataset** - lista os vendedores e a localização de cada.
* **product_category_name_translation** - tradução para o inglês dos nomes das categorias de produtos, presentes no arquivo olist_products_dataset em português.

O esquema de dados original possui a seguinte composição:

![Modelo de dados da Olist](image/modelo_de_dados_original.png)


## ETL no Databricks:
O Databricks foi escolhido por ser uma plataforma de notebooks, que suporta diversas linguagens de programação e permite a criação de fluxos de trabalho unificados e automatizados, o que auxilia o processo de modelagem de dados. Com isso, foi construído um padrão de design de dados conhecido como "arquitetura medalhão", que é usado para direcionar logicamente os dados, com o intuito de individualizar as principais etapas e realizar uma melhoria incremental e progressiva, à medida que o dado flue através de cada camada. As camadas são definidas como Bronze, Silver e Gold, e são equivalentes as camadas conhecidas por Raw, Stage e Final, respectivamente, que são nomenclaturas já popularizadas em projetos de dados. 

A descrição do que foi feito em cada camada é apresentada a seguir, com uma explicação inicial do passo a passo tomado e das decisões aplicadas, seja da extração dos dados de origem, do tratamento e limpeza dos dados ou da modelagem definida. Vale a pena ressaltar que explicações mais detalhadas do código se encontram ao longo dos notebooks, por meio de comentários e títulos das células de código.


### 1. Camada Bronze

A camada bronze é responsável por transformar em tabelas os arquivos csv do dataset, que foram previamente importados para o storage do Databricks. Para isso, criamos um banco de dados nomeado de olist_bronze e executamos um código em Python que vai rodar em loop, fazendo a leitura de cada arquivo até que todas as tabelas sejam criadas. Também foi adicionado o código de execução individual de cada tabela, para caso se necessite fazer desta forma. O notebook pode ser executado via botão "Run all", para a opção de criação das tabelas automaticamente pelo loop, e irá parar ao chegar no comando de exit. A imagem abaixo ilustra o catálogo de dados com as tabelas criadas ao fim da execução, uma para cada arquivo de origem.

![Tabelas Camada Bronze](image/database_camada_bronze.png)


### 2. Camada Silver
A camada silver é responsável por realizar todo o tratamento e limpeza dos dados. Nela, temos a criação do banco de dados olist_silver e tratamos cada arquivo oriundo da camada bronze individualmente, passando por cada campo analisando quais tratamentos serão necessários. Essas análises se dão por consultas em SQL, com uma investigação minunciosa do dado de origem a procura de anomalias, erro de formatação, erro de escrita, tipo de dado inapropriado, eliminação de nulos e realização de "de-paras", quando se faz necessário. Por fim, é criada a tabela stage com todos os tratamentos aplicados e alguns relacionamentos entre tabelas feitos. A imagem a seguir ilustra as tabelas criadas na camada silver.

![Tabelas Camada Silver](image/database_camada_silver.png)


### 3. Camada Gold
Por fim, a camada gold é responsável pela criação das tabelas fato e dimensão, que compõem o schema estrela que pretendemos criar. A partir da criação do banco de dados olist_gold, são criadas cada tabela ft (fato) ou dm (dimensão), conforme sua configuração no modelo de dados, e realizados alguns testes de validação, como o de identificação de id's repetidos, o que é de suma importância para criarmos as chaves primárias únicas. Também é nesta camada que novas configurações de tabelas são realizadas, a partir da normalização dos dados, construção de novos relacionamentos, e eliminação de dados que não terão uso, tudo para que vá se construindo uma modelagem no formato estrela ou snowflake, que são os schemas mais apropriados para uso no Power BI, segundo recomendação da plataforma. Por último, temos uma seção extra que foi criada para a realização de testes de homologação dos dados apresentados no Power BI, para que se tenha uma validação das métricas, KPI's e gráficos construídos. A imagem a seguir ilustra as tabelas fato e dimensão criadas na camada gold.

![Tabelas Camada Gold](image/database_camada_gold.png)


Após a finalização das três camadas de dados no Databricks, foram importadas todas as tabelas da última camada (Gold) para dentro do Power BI, para que se inicie a próxima etapa, a construção do Data Visualization.


## Data Visualization no Power BI

Para a construção do dashboard, foi realizado um investigação dos dados de origem afim de mapear quais indicadores e métricas poderiam ser extraídos. Quais perguntas gostaríamos de responder a partir da análise desses dados. Tendo por guia essa narrativa construída, deu-se início a construção dos visuais do dashbord. Seja pela escolha dos gráficos mais apropriados para uma leitura mais clara e direta sobre o que precisava ser informado, ou os totalizadores das dimensões ou até mesmo a segmentação dos dados. 

No Power BI, foi necessário realizar um ajuste no modelo dimensional. A tabela de location se relacionava com duas tabelas dimensões diferentes: customers e sellers. Já a tabela datetime tinha vários relacionamento diferentes com a tabela fato orders, e outros dois com order_reviews. Essa conjuntura de apenas uma tabela datetime e location realizando múltiplos relacionamentos ocasionaria um grande problema para os nossos resultados, uma vez que ao realizar um filtro outras tabelas indesejadas seriam também impactadas, dado que estariam todas interligadas. A partir disso, as tabelas location e datetime foram duplicadas dentro do Power BI, criando uma tabela única para cada relacionamento que se necessite fazer. O modelo de dados final, em schema snowflake, pode ser visualizado pela imagem abaixo.


![Diagram do modelo de dados](image/diagrama_modelo_de_dados.png)


Para mais informações sobre o tema, acesse o [artigo](https://learn.microsoft.com/pt-br/power-bi/guidance/star-schema) da Microsoft que relata sobre dimensões com função múltipla.
No Power BI foram construídas duas páginas, na primeira temos um one-page com várias leituras gráficas e na segunda página temos uma nuvem de palavras que representa as palavras com mais frequência dentro das avaliações dos clientes.

#### Apresentação do Dashboard

O one-page apresenta 5 filtros de segmentação de dados, em que dois são relacionados ao tempo e três a localidade. Estes filtros podem ser utilizados individualmente ou em conjunto. Na barra lateral esquerda é possível visualizar cartões com totais de dados para algumas das dimensões mais importantes. Por exemplo, o cartão de Faturamento nos informa o total geral das vendas realizadas em toda nossa base.

O primeiro gráfico "Total de vendas por Mês" nos apresenta o quantitativo de vendas realizadas ao longo dos meses, em ordem decrescente, tendo o mês de Agosto como o mês de maior volume de vendas, mas também bem próximo do segundo colocado Maio. Aqui podemos supor a influência das datas comemorativas de dia dos pais (segundo domingo de Agosto) e dia das mães (segundo domingo de Maio), por exemplo. Porém pode ser também por alguma iniciativa interna da loja, como promoção ou melhoria de estoque.

O segundo gráfico "Top 10 total de vendas por categoria" nos apresenta o mesmo quantitativo de vendas que o anterior, porém agora para as 10 categorias de produtos com maior número de vendas. Podemos observar que as categorias de cama_mesa_banho, beleza_saude e esporte_lazer são as que mais têm aderência do consumidor. Essa métrica pode indicar a loja quais categorias ela deve investir para aumento de estoque ou quais categorias não estão com bom número de vendas.

O gráfico "Avaliação dos Clientes" apresenta a distribuição das notas de avaliação dos clientes, com o quantitativo de cada uma das notas (1 a 5). É possível afirmar que a maioria dos clientes que avaliou a compra está satisfeita com o serviço e produtos oferecidos. Mas o dado também nos alerta para a segunda coluna com maior quantidade de notas, que é a nota mais baixa (1), indicando que mesmo com bons resultados, há uma parcela significativa de clientes que ficaram extremamente insatisfeitos com sua compra.

O gráfico "Tipo de Pagamento" nos apresenta quais os meios de pagamento mais utilizados pelos clientes. Em primeiro lugar, e de forma bastante expressiva, temos o cartão de crédito que é o grande campeão da preferência, com uma média de 3,51 de números de parcelas escolhidas para quando é decidido por parcelamento de compras. Esse dado pode beneficiar a loja com impulsionamento de estratégias sobre juros no cartão de crédito, melhoria de parcelamentos ou até mesmo oferecer benefícios de desconto para incentivo de pagamentos à vista no boleto ou no cartão de débito.

Por fim, temos o mapa geográfico de "Distribuição de Vendas por Estado" que apresenta o total de vendas de acordo com a localidade. Este gráfico utiliza saturação de cor para representar as quantidades, onde os valores mais altos possuem tons mais fortes de azul e os valores mais baixos possuem um azul bem claro tendendo ao cinza. Ao passar o mouse por cada estado é possível visualizar a contagem de vendas referente. Nos dois cartões abaixo do mapa temos o total de cidades e estados abrangidos.

![One-page report](image/dashboard_pag1.png)

A segunda página nos apresenta dois visuais de nuvem de palavras das avaliações dos clientes, um para avaliações negativas e outro para as positivas. Este visual passou por um filtro de "palavras irrelevantes", que nada mais são do que palavras que não nos traz informações válidas, como preposições e pronomes. As palavras com mais repetições são as que aparecem em maior destaque de tamanho. Esta é uma forma de visualização simples e prática que nos ajuda a monitar os apontamentos dos clientes.

![One-page report](image/dashboard_pag2.png)

