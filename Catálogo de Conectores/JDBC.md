# Criação de pipelines para fontes JDBC

Nesta seção você encontra as instruções para a criação de uma DAG de extração de fontes com protocolo JDBC (Java™ Database Connectivity).
  
  
## Integração
	
 A integração ``JDBC`` é realizada por meio do método ``read`` do Spark. Utilizamos a consulta via *query*, montada de acordo com as opções selecionadas pelo ``usuário``, como explicaremos abaixo. Para entender melhor o funcionamento desse tipo de integração, acesse esta [**Documentação**](https://spark.apache.org/docs/latest/sql-data-sources-jdbc.html).


## Atributos das *tasks*
	
As ``tasks`` na ingestão de ``JDBC`` correspondem às tabelas a serem ``ingeridas``. No qual cada ``task`` corresponde a uma tabela e todas as tabelas compõem uma ``DAG`` do mesmo sistema de origem que utilizam o mesmo intervalo de ``execução``.

Os atributos das tabelas, diferentemente dos atributos da ``DAG``, podem variar de acordo com a estratégia pretendida para a ``ingestão`` de cada tabela. Por isso, é necessário se atentar à composição dos atributos ``preenchidos`` para cada tabela.

~~~yaml
datasets:
  - name: 
  	domain: 
    entity: 
    task_owner: 
    owner_team: 
    active:  
    schema: 
    database_type: 
    columns: 
    connection_id: 
    notifier:
     
~~~
---
- ``name``: é o nome da tabela. Ele deve ser ``preenchido`` exatamente da maneira como está no ``banco`` de dados, pois servirá como parte da ``identificação`` da tabela na ``query`` de consulta no banco de dados. Caso contrário, a task não funcionará.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``domain``: Dominio de negocio em que a fila esta representada.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``entity``: Entidade de negocio em que a fila esta representada.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``task_owner``: é a pessoa ``responsável`` que vai cuidar da task.

  - ``Validação``: obrigatório, não nulo e string.
---
- ``owner_team``: é o time do ``responsável`` pela task. 

  - ``Validação``: obrigatório, não nulo, string e CamelCase.
---

- ``active``: é para saber se sua task vai estar ``ativa`` ou ``não`` no caso ``habilitar`` e ``desabilitar`` a task.

  - ``Validação``: obrigatório, ``True`` ou ``False``
---
- ``schema``: é o schema no banco de dados onde está ``inserida`` a tabela. Assim como o ``name``, deve ser escrito ``exatamente`` da maneira como está no banco de dados.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``database_type``: aqui deve ser informado o tipo de banco de dados ao qual se realizará a conexão. Atualmente estão disponíveis: ``sqlserver``, ``postgresql`` e ``mysql``.

  - ``Validação``: obrigatório, não nulo e string. Aceito apenas: "sqlserver", "postgresql", "mysql".
---
- ``columns``: são as colunas que serão ``extraídas``. Há a opcão de passar apenas as colunas pretendidas ou um **"*"** para selecionar todas. Esse campos será passado direto no SELECT no banco, ou seja, devem ser passadas instruções que o banco ``aceite``.

  - ``Validação``: obrigatório, não nulo e string.
---
- ``connection_id``: o nome da ``connection`` criada na UI do ``airflow`` com as credenciais para a conexão com a fonte.

  - ``Validação``: obrigatório, não nulo, string e snake_case.
---
- ``notifier``: É onde você configura para notifica-lo sobre as ações na DAG.

  - ``Validação``: obrigatório, não nulo e dict. type aceito apenas: e-mail e ms_teams.
  - ``Exemplo``:   
 ~~~yaml
       notifier:
        type: "ms_teams"
        receivers:
          - 'IrisDataGovernanceWebhook'
~~~

O processo de preenchimento do **Webhook** para o Notifier é realizado por meio do **Key Vault**, um recurso altamente **seguro** e **eficaz**. Consulte um documentação específica que elaboramos para ajudá-lo nessa tarefa. [**Notifier Teams Webhook**](/iris-ingestion/2-0-architecture/notifier-webhook)

---

- Para saber mais sobre os atributos da Task consulte nossa [**Documentação**](/iris-ingestion/getting-started/creating-dag/task-parameters).
## Atributos obrigatórios
  São os atributos **obrigatórios**, que toda tabelas deve ter:
~~~yaml
datasets:
  - name: 
  	domain: 
    entity: 
    schema: 
    task_owner: 
    owner_team: 
    active: 
    database_type: 
    columns: 
    connection_id: 
    notifier:    
~~~

## Yaml Exemplo - JDBC(SqlServer)

Exemplo, de como seu <kbd>.yaml</kbd> deve ficar:
~~~yaml
dag:
  dag_type: 'sqlserver'
  dag_class: 'source'
  system: 'mes_global'
  country: 'Brazil'
  schedule: '0 7 * * 1'
  datasets:
    - name: 'tSample'
    	domain": 'GlobalLM'        
      entity": 'Sample'
      schema: 'dbo'
      task_owner: 'Thauany Rocha - BRPGM0063@ambev.com'
      owner_team: 'Iris'
      active: True,
      database_type: 'sqlserver',
      columns: '*'
      connection_id: 'jdbc_mes_global'
      metadata: !include ../metadata/mes_global_tsample.yaml
      notifier:
      	type: 'ms_teams'
        receivers:
          - 'SEU_TOKEN_DO_TEAMS_PARA_RECEBER_MSGS'
~~~

## Atributos variáveis e estratégia de ingestão
	
 Para cada estratégia de ingestão pretendida para a tabela, um conjunto de argumentos devem ser passados para obter o resultado ``pretendido``. É necessário se atentar muito à estratégia, pois ela envolve os argumentos da task em ``questão``, das outras tasks e até mesmo da ``DAG``.
___
  
**Full table**

Nessa estratégia, nenhum filtro é passado, apenas as colunas são ``selecionadas`` e a ``query`` no banco retornará a tabela toda.

``Não`` serão realizadas execuções retroativas, exceto na hipótese de alguma outra tabela da mesma ``DAG`` optar pela estratégia de ``backfill``. Por isso, o ``start_date`` anterior à data atual somente iniciará a sua execução em D+*scheduler_interval*. Caso alguma outra tabela opte por fazer ``backfill`` é necessário preencher uma data presente ou futura no ``tart_date`` da tabela que opte por ``full table``, caso contrário, diversas execuções serão realizadas na ``full table``, retornando o mesmo dataset em todas, o que é ``desnecessário``.

``Essa estratégia é recomendada para tabelas médias ou pequenas.``
  
~~~yaml
datasets:
  - name:
  	domain:
    entity:
    schema:
    task_owner:
    active:
    database_type:
    owner_team:
    start_date:
    columns:
    connection_id:
    notifier: 
~~~
---

**Filter**
	A segunda opção é a extração com ``filtro`` Nessa, um parâmetro de ``filtro`` é passado e servirá de ``where``na *query*, por isso, deve ser um ``filtro`` aceito pelo *sparksql*. O ideal é que esse tipo de estratégia seja realizada em conjunto com o ``sync_mode`` "append", a fim de que cada execução insira no data lake um conjunto de dados complementar ao já existente, formando um conjunto da soma de todas as ingestões na ``rawzone``.
Nese cenário, serve ainda a regra explicada acima a respeito da *start_date*: é importante que ela seja definida com data **atual** ou **futura**.
Essa estratégia é interessante para tabelas de sistemas em que os dados podem ser alterados em uma certa janela de tempo. Dessa foram, caso o filtro contemple da data atual, menos a janela de alteração, tudo isso comparado com alguma coluna de data de criação do registro, será possível trazer sempre o dataset com as **alterações** e ir somando com os dados já existentes no data lake.
 
~~~yaml
datasets:
  - name:
  	domain:
    entity:
    schema:
    task_owner:
    active:
    database_type:
    owner_team:
    start_date:
    columns:
    filter: "isert_date >= date_sub(current_date(), 5)"
    connection_id:
    notifier:
~~~
---

**Backfill**
O *backfill* é a ingestão com execuções **retroativas**. Adotada essa estratégia, o airflow irá disparar diversas execuções da ``DAG``, a partir do *start_date* da tabela - desde que ele seja igual ou posterior ao da DAG, caso contrário, considerará o da ``DAG`` - somando o *scheduler_interval*, olhando para uma coluna de referência para fazer o filtro.
Ou seja, é possível fazer uma ingestão de uma tabela olhando para uma data do **passado** e ir **trazendo** os dados correspondentes àquele intervalo de execução, até chegar à data corrente.
  
**Exemplo:**
![backfill.jpg](/processo_de_desenvolvimento_arq_2/backfill.jpg)

- ``start_date`` - data de início da **Execution date**. No exemplo: 2019-01-01.
- ``execution date`` - data de referência da execução.
- ``start date`` - data real de início daquela execução.
- ``scheduler_interval`` - intervalo entre as **Execution date**. Neste caso, "daily".
- ``backfill`` - execuções retroativas.
- ``next execution date`` - próxima execução.
	
Ou seja: um **filtro** é feito para que cada execução traga apenas os dados referentes à data entre a **execution date** anterior e a atual, baseando-se em uma ``filter_column``, que deve ser do tipo *date*.
  
Sendo assim, a primeira execução do exemplo trará os dados do dia 2019-01-01; a segunda trará os dados do dia 2019-01-02; a terceira os dados do dia 2019-01-03 e assim sucessivamente. A partir de então, as **DAGs** somente serão acionadas quando a próxima **execution date** chegar.

Como foi dito acima, uma vez que uma tabela da DAG use *backfill*, todas as tasks que tiverem *start_date* anterior à data atual também farão execuções **retroativas**, mas trarão o dataset todo (estratégia full table) ou o dataset filtrado (estratégia filtro), por isso, adeque o **start_date** da tabela de acordo com a diretriz da estratégia escolhida.

É importante ressaltar que tal estratégia deve ser usada com o ``sync_mode`` "append" para que todos os datasets correspondentes ao respectivo intervalo trazido sejam somados aos outros. Além disso, como já foi dito, é necessário que a **start_date** da DAG seja igual ou anterior à **start_date** da task com *backfill*. Por fim, tal estratégia é recomendada para carga de histórico de datasets grandes nos quais a execução de uma query grande, sem filtro, é inviável.

~~~yaml
datasets:
  - name:
  	domain:
    entity:
    schema:
    task_owner:
    active:
    database_type:
    owner_team:
    start_date:
    columns:
    backfill:
			filter_column: "insert_date"
    connection_id:
    notifier: 
~~~