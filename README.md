# O que é

A Arquitetura 2.0 torna o usuário final o principal orquestrador dos processos de ingestão. Neste cenário, contamos com ferramentas de gerenciamento de processos apartados da execução dos trabalhos em si. 
Em outras palavras, utilizamos o Apache Airflow para agendar os jobs e o Databricks para executar os processos de Ingestão. Quem gerencia o processo, da origem até o fim, é o usuário que conhece a fonte, através de uma dag que é customizada no GitHub. Na seção abaixo, explicamos mais sobre os papéis e responsabilidades dos integrantes.

## Papéis e responsabilidades 

Tarefas de manutenção coletiva (saúde ambiente de desenvolvimento, implementação de novas features, etc) têm como responsáveis o time de Ingestion. Entretanto, é importante ressaltar que há também responsabilidades dos próprios criadores de pipelines.

### O que fazemos
<font color='green'>➔</font> Garantir estabilidade dos   conectores de dados.
<font color='green'>➔</font> Desenho, implementação e monitoramento de arquitetura adequada para Big Data.
<font color='green'>➔</font> Manutenção do Data Lake.
<font color='green'>➔</font> Operação e manutenção de todas as ferramentas.
<font color='green'>➔</font> Compliance LGPD
<font color='green'>➔</font> Garantir que os processos sejam executados conforme padroes
<font color='green'>➔</font> Evolução da plataforma.

### O que não fazemos
<font color='red'>➔</font> Manutenção em DAGs de ingestão.
<font color='red'>➔</font> Acompanhamento da execução de DAGs.
<font color='red'>➔</font> Desenvolvimento de documentação dos datasets e modelos.
<font color='red'>➔</font> Modelagem de dados.

# Arquitetura e Infraestrutura

Saiba mais sobre como foi modelada a Arquitetura, bem como a regra de negócio para o transporte dos dados entre a Rawzone e Trustedzone. 


📑 [**Tutorial - Arquitetura e Infraestrutura 2.0**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/tutorial/infrastructure-architechture)

✈ [**Explanation - Utilizando uma arquitetura híbrida**](https://iris.ambev.com.br/pt-br/data-transformation/explanation/hybrid-architecture)

# Pré-requisitos para a Ingestão 

É de extrema importância se certificar de que os devidos acessos foram solicitados, bem como realizar a instalação das ferramentas em sua máquina, respeitando a versão e etapas do processo. Também deve-se garantir de que o dado ser enviado ao Lake já não é alimentado atualmente.
Para isso, consulte no link abaixo como realizar os processos obrigatórios:

❓ [**How to - Atividades obrigatórias pré ingestão**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/access)

# Componentes



## Airflow

O Apache Airflow é uma plataforma de gerenciamento e agendamento de processos, também chamados de DAGs. A DAG é composta por uma ou mais tasks, sendo estas responsáveis por executar o job que irá ser executado no ambiente. Nesse contexto,cada DAG comporta uma ou várias tasks e, cada task comporta somente um dataset.

> [**Conheça nosso catálogo de conectores**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/ingest#cat%C3%A1logo-de-conectores)
{.is-info}

❓ [**How to - Crie DAGs no Airflow**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/ingest)
❓ [**How to - Migração de dados históricos**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/migracao-dados-historicos)
✈ [**Explanation - Estratégias de transformações**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/transformations-ingestion)
✈ [**Explanation - Caminhos dinâmicos**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/dynamic-paths)
✈ [**Explanation - Criando DAGs mais eficientes: Pools de Execução**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/dag-pools)
✈ [**Explanation - Serviços de Sincronia (Sync Service)**](https://iris.ambev.com.br/pt-br/data-transformation/explanation/syncservice)
✈ [**Explanation - Conector de API: Anaplan**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/connectors/rest-api#estrat%C3%A9gia-de-api-conex%C3%A3o-com-anaplan)

## Databricks

Na Iris Ingestion, o Databricks é utilizado para processamento e movimentação dos dados entre zonas de armazenamento do Data Lake. Também é possível acompanhar o log das execuções dos jobs e validar os dados transportados. No link abaixo é possível entender como é feita a validação dos dados.

❓ [**How to - Explore e valide os dados**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/databricks)
📑 [**Tutorial - Prevenir a corrupção dos dados**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/tutorial/data-anti-corruption)

## Monitoramento e notificações

O monitoramento e notificação dos processos em Ingestion permitem que se assegure a entrega dos dados e, em casos de falha, que haja previsibilidade.
Sendo assim, a arquitetura 2.0 conta com diversos monitores dos ambientes (Airflow e Databricks) e das DAGs em execução. Também é possível configurar alertas no Teams em caso de falhas. Confira abaixo as etapas para estes processos.

✈ [**How-to - Crie notificações no Teams e no DataDog**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/notifications-monitoring)
✈ [**Explanation - Métricas de sucesso**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/success-metrics)


# Dúvidas? 
Em caso de dúvidas entre em contato conosco através do nosso  <a href="https://dev.azure.com/AMBEV-SA/IRIS-Analytics%20Platform/_boards/board/t/AnalyticsTeam/Customer%20Service" target="_blank" >
