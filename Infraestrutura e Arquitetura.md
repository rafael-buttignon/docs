# Infraestrutura


![infra_ingestion.gif](/iris-ingestion/infra_iris_ingestion.gif)

O fluxograma acima apresenta o caminho dos dados, desde o momento em que ele está na fonte (*Pre-built readers*) até a etapa final de catalogação. A forma pela qual o dado será ingerido é via *conector*. Este conector faz a autenticação no serviço de origem, o tratamento inicial e a final para a <kbd>Raw Zone</kbd> e <kbd>Trusted Zone</kbd> respectivamente, utilizando o **Data Lake Storage** para armazenamento. Por fim, o schema é catalogado no Metabase. 

O conector será uma *task/dataset* que será executada no ambiente do **Airflow**, por meio do operador <kbd>DatabricksSubmitRunOperator</kbd>. Com ele, o Airflow irá agendar o início da task que se conecta na máquina **Databricks** correspondente ao conector e performa as operações de ingestão com o auxílio do **Apache Spark**.

> Atualmente temos um ambiente de Prod (de uso dos usuários) e um de nonProd. O ambiente de nonProd é restrito para o time de desenvolvimento da Ingestion. Isso permite que possamos implementar e testar as novas features de forma apartada e controlada antes que o produto venha a produção.
{.is-danger}


## Apache Airflow

O Airflow é um ambiente de agendamento de tarefas, tornando possível o gerenciamento de **Grafos Acíclicos Direcionados (DAGs)**. Cada DAG comporta uma ou várias tasks e, cada task comporta somente um dataset. Isso significa que na mesma DAG é possível adicionar várias ingestões de API, por exemplo, sendo cada ingestão em uma task diferente. Caso exista uma nova demanda utilizando um conector diferente, é necessário criar uma nova DAG. 

As DAGs tomam como base as configurações passadas no arquivo **YAML** que, com o auxílio do repositório **Correnteza** e a biblioteca **Pyiris**, irão gerar o formato final a ser carregado no ambiente do Airflow. As configurações das DAGs podem ser atualizadas posteriormente e tomam como base os parâmetros nativos do Airflow. As informações relacionadas ao conector são parametrizadas de acordo com as documentações da Íris. 
 
## Pyiris
 
[**PyIris**](https://github.com/cervejaria-ambev/pyiris) é o repositório central onde ficam todas as integrações com as fontes de dados para leitura, escrita, monitoramento, governança e onde também são construidas todas as abstrações de transformação de dados.


## Correnteza
  
[**Correnteza**](https://github.com/cervejaria-ambev/correnteza) é o repositório responsável por abstrair a criação e personalização de todos os **fluxos** de trabalho de uma DAG para qualquer tipo de pipeline de dados. 

  
## Azure Data Lake Storage Gen2
O Azure Data Lake Storage Gen2 é um conjunto de recursos dedicados à análise de **Big Data**, criado sobre o armazenamento de arquivos do Azure que permite o fácil gerenciamento de grandes quantidades de dados, com desempenho otimizado, segurança e baixo custo. Na Iris Ingestion esse serviço é utilizado para o **armazenamento** de todos os tipo de dados da companhia.
 
O Storage V2 é um tipo de conta de armazenamento básico para blobs, arquivos, filas e tabelas. Na plataforma é utilizado como uma zona de armazenamento temporário dos dados que não podem ser convertidos direto pelo Airflow em Delta (formato utilizado para os arquivos na <kbd>Trusted Zone</kbd>). Os dados temporários são apagados após o processo de transporte do **Data Lake**.

Devido ao fato do Delta permitir mais funcionalidades e performance de leitura e escrita, tal estratégia permite que possamos nos ajustar à arquitetura do Global, bem como respeitar os conceitos de Data Mesh, respeitando os padrões de zona e domínios.
 
Consulte mais informações sobre esta ferramenta [**clicando aqui**](https://docs.microsoft.com/pt-br/azure/storage/blobs/)
 
## Azure Databricks 
O Azure Databricks é uma plataforma de análise **rápida**, fácil e colaborativa baseada no Apache Spark e otimizada para a plataforma de serviços de nuvem do Microsoft Azure. A ferramenta abrange as tecnologias e os recursos completos de código aberto do cluster Apache Spark e tem um ecossistema variado com Spark SQL, dataframes, streaming, machine learning, computação gráfica e suporte a diferentes **linguagens** (R, SQL, Python, Scala e Java). Na Iris Ingestion, o Databricks é utilizado para processamento e movimentação dos dados entre zonas de armazenamento do Data Lake.
 
Consulte mais informações sobre esta ferramenta [**clicando aqui**](https://docs.microsoft.com/pt-br/azure/azure-databricks/)      
  
# Arquitetura
A arquitetura 2.0 foi desenhada pra seguir alguns princípios do **DDD (Domain-Driven Design)** e do ***Data Mesh***. O *DDD* preza pela ideia de que o desenvolvimento tenha como base o domínio do negócio, tornando o usuário participante de todo o processo de planejamento, implementação e validação do projeto. O Data Mesh, por sua vez, torna as equipes de negócios as responsáveis pelos dados gerados. Embora os dados não fiquem centralizados, isso permite que os times tenham mais autonomia sobre os dados que eles mesmos geram.

Na nossa realidade, a arquitetura da ingestão descentraliza a responsabilidade do dado, e torna as equipes de negócio/domínio protagonistas e responsáveis pelas informações a serem transformadas. Isto é, o usuário que gera o dado, também faz sua ingestão no ambiente e garante que o mesmo é processado com sucesso. 

A ideia central é que os times passem a trabalhar, entender e criar uma separação lógica de como devem ser implementados os domínios e as entidades das fontes **(source)** e analíticos **(analytical)** dos seus dados.

  ![infra_data_modeling.gif](/iris-ingestion/infra_data_modeling.gif)


A ingestão de dados acontece no ambiente do Airflow, onde cada DAG será responsável por um domínio e módulo. A DAG irá conter um ou mais contextos, utilizando as Tasks para cada fonte (*dataset*) consolidar as informações de acordo com seu módulo. Cada execução da Task irá extrair os dados, trata-los e aplicar regras de governança, nesta ordem.

- **Extração dos Dados**
Cada DAG irá utilizar um módulo (conexão) específico e, dentro desta, cada Task irá extrair os dados de um dataset específico. A etapa de extração tomará o conector como protocolo base para estabelecer a comunicação com a fonte e, por fim, trazer os dados em formato DataFrame através do Apache Spark.

- **Tratamento dos Dados**
Após a extração, os dados são tratados de forma primária e levados para a Raw Zone, camada mais superficial de organização dos dados. Em um segundo momento, os dados serão tratados com etapas de de transformação, como: mascaramento, remoção de valores duplicados, formatação de colunas, entre outras regras customizadas para cada realidade de negócio. Após isso, estes dados serão dispostos na Trusted Zone, camada de informações tratadas.

- **Governança dos Dados**
Finalmente, será aplicada regras de governança nos dados tratados, e os metadados (*schema*) relacionados ao domínio e fonte serão dispostos no Metabase para consulta e validação.

> Nesta nova arquitetura, todos os dados respeitam um padrão de nomenclatura que difere da versão 1.0. Por isso, em caso de novas migrações, pode ser necessário reapontar as dependências dos serviços.