# Criação de pipelines para fontes RabbitMQ
 
Nesta seção você encontrara as instruções para a criação de uma DAG de extração de fontes com conexão AMQP no RabbitMQ.
 
## Integração
A integração com o ``RabbitMQ`` é feita através da biblioteca do Python chamada de [**Pyka**](https://github.com/pika/pika). Utilizamos o método ``consume``dessa bibloteca para se conectar a uma fila do RabbitMQ via protocolo AMQP para puxar todas as mensagens ``persistidas`` na fila.

## Atributos das *tasks*
	
As ***tasks***, na ingestão do **RabbitMQ**, correspondem às filas a serem ingeridas. Cada *task* é uma fila. Todas as filas que compõem uma DAG são do mesmo **serviço** de RabbitMQ e utilizam o mesmo **intervalo de execução**.

Os atributos das filas, diferentemente dos atributos da DAG, podem variar de acordo com a estratégia pretendida para a ingestão de cada tabela. Por isso, é necessário se atentar à composição dos atributos preenchidos para cada tabela.

~~~yaml
datasets:
  - name: 
  	domain: 
    entity: 
    task_owner: 
    owner_team: 
    active:  
    execution_timeout:
    prefetch_count:
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
- ``execution_timeout``: timeout de execução da conexão como rabbitMQ em segundos.
	- ``Validação``: opcional, não nulo e integer.
---
- ``prefetch_count``: limitador de mensagens a serem capturadas no rabbitMQ.
	- ``Validação``: opcional, não nulo, integer e menor que 60000.
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
São os atributos **obrigatórios**, que toda tabela deve ter:
~~~yaml
datasets:
    - name: 
      domain: 
      entity:
      task_owner:
      owner_team: 
      active:
      execution_timeout:
      prefetch_count:
      connection_id:
      notifier:
~~~



## Yaml Exemplo - RabbitMQ 

Exemplo, de como seu <kbd>.yaml</kbd> deve ficar:
~~~yaml
dag:
	dag_type: 'rabbitmq'
  dag_class: 'source'
  system: 'Bifrost'
  country: 'Brazil'
  schedule: '0 7 * * 1'
  datasets:
    - name: 'IRIS_ORDER_SITE'
      domain: 'Order'
      entity: 'OrderSite'
      task_owner: 'Thauany Rocha - BRPGM0063@ambev.com'
      owner_team: 'Sales'
		  active: True
      execution_timeout: 450
      prefetch_count: 30000
      connection_id: 'rabbitmq_bifrost'
      metadata: !include ../metadata/order_site_contract.yaml
      notifier:
      	type: 'ms_teams'
        receivers:
        	- 'SEU_TOKEN_DO_TEAMS_PARA_RECEBER_MSGS'
~~~

<br>

## Estratégia de inserção

É importante ``frizar`` que como o RabbitMQ persiste as mensagens da fila conforme os eventos são sendo gerados nos sistemas origens, a estratégia de insercão utilizada para **pipelines** são sempre de utilizar o modo ``append``, ou seja, todo dado que é consumido das filas são appendados nos arquivos ja existentes, mantendo a **imutabilidade** no dados, sem a necessidade de se utilizar estratégias mais complexas de inserção.

Também ressaltamos que o uso de ``overwrite`` para conectores de mensageira não é recomendável, pois a cada nova interação com a fila do conector de mensageira, a ação de sobrescrever iria substituir os dados atuais no data lake. Portanto, é sempre importante pensar em um caso de uso no qual isso seja benéfico para a sua regra de negócio. Como as mensageiras operam quase em tempo real, não seria correto sobrescrever os dados. Assim, no contexto de uso de mensageiras, a utilização de overwrite não é recomendada.

---
 