# Criação de pipelines para fontes Service Bus

Nesta seção você encontrarar as instruções para a criação de uma **DAG** de extração de fontes com conexão Service Bus.

## Integração
A integração com o ``Service Bus`` é feita através da biblioteca do Python chamada de [**Azure-Servicebus**](https://pypi.org/project/azure-servicebus/). Utilizamos os método principais dessa biblioteca para se conectar a uma fila do Service Bus para puxar todas as mensagens e **persistidas** na fila.

## Atributos das *tasks*
	
As ``tasks``, na ingestão do ``Service Bus``, correspondem às filas a serem ingeridas. Cada ``task`` é uma fila. Todas as filas que compõem uma DAG são do mesmo serviço de ``ervice Bus`` e utilizam o mesmo ``intervalo de execução``.

Os atributos das filas, diferentemente dos atributos da DAG, podem variar de acordo com a estratégia pretendida para a ingestão de cada tabela. Por isso, é necessário se atentar à composição dos atributos preenchidos para cada tabela.

~~~yaml
datasets:
    - name: 
    	domain: 
      entity:
      task_owner:
      owner_team: 
      active: 
      debug:
      max_message_count: 
      max_wait_time: 
      prefetch_count:
      batch_size_to_persist:
      execution_timeout:
      max_lock_renewal_duration:
      connection_id:
      notifier:
      
~~~
---
- ``name``: Certifique-se de configurar esse parâmetro com o nome exato da fila desejada para consumo. Preencha o parâmetro com o nome da sua fila de origem, exatamente como é.

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
- ``max_message_count``: limitador de mensagens a serem capturadas no Service Bus. max_message_count declara o número máximo de mensagens para tentar receber antes de atingir o limite de bloqueio de mensagem da fila, que por padrão é (5 minutos).

-   *É importante destacar que esse parâmetro irá funcionar de acordo com o esquema (schema) da sua fila para ingestão. Se o seu esquema for complexo e extenso, é recomendado reduzir o consumo para evitar erros e problemas relacionados à duração da fila. Certifique-se de ajustar os parâmetros de acordo com a complexidade do esquema e o ritmo de consumo da fila.*

	- ``Validação``: opcional, não nulo, integer e menor ou igual a 100000.
---
- ``max_wait_time``: Tempo de espera, que a conexão vai esperar chegar alguma mensagem, caso não tenha mensagem ou a mensagem acabou, fechara a conexão automaticamente, padrão 30 Segundos. **(Não recomendamos mudar esse parametro!)**

	- ``Validação``: opcional, não nulo, integer, entre (5 segundos a 120 segundos)
  
---
  - ``prefetch_count``: O número máximo de mensagens a serem armazenadas em cache com cada solicitação ao serviço. Essa configuração é apenas para ajuste de desempenho avançado. Aumentar esse valor melhorará a taxa de transferência de mensagens, mas aumentam a chance de que as mensagens expirem enquanto estiverem em cache, se não estiverem. **(Não recomendamos a mudar esse parametro!)**

	- ``Validação``: opcional, não nulo, integer, entre (0 á 2000)
---
- ``max_lock_renewal_duration``: É importante ressaltar que o bloqueio da mensagem deve ser renovado no tempo máximo de 5 minutos. Recomendo solicitar ao criador da fila que defina o tempo de bloqueio para 5 minutos e escolha um valor inferior a esse para a renovação, como por exemplo 2 minutos. Dessa forma, a cada 2 minutos todas as mensagens serão renovadas.

	- ``Validação``: opcional, não nulo, integer, entre (0 á 500)
---
- ``execution_timeout``: O parâmetro 'execution_timeout' determina o tempo durante o qual a conexão permanecerá buscando mensagens. Por padrão, a configuração é de 5 minutos. Embora seja possível modificar esse parâmetro, é aconselhável estabelecer um padrão para a ingestão do seu conjunto de dados.

	- ``Validação``: opcional, não nulo, integer, entre (60 á 3600)
---
- ``batch_size_to_persist``: Defina o tamanho máximo do lote para cada solicitação na fila. A cada requisição na fila, será criado um lote com a quantidade especificada no parâmetro.

-   *É importante destacar que esse parâmetro irá funcionar de acordo com o esquema (schema) da sua fila para ingestão. Se o seu esquema for complexo e extenso, é recomendado reduzir o consumo para evitar erros e problemas relacionados à duração da fila. Certifique-se de ajustar os parâmetros de acordo com a complexidade do esquema e o ritmo de consumo da fila.*

	- ``Validação``: opcional, não nulo, integer, entre (1500 a 30000)
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
      connection_id:
      notifier:
      
~~~
 
 ## Yaml Exemplo - Service Bus

Exemplo, de como seu <kbd>.yaml</kbd> deve ficar:

~~~yaml
dag:
	dag_type: 'servicebus'
  dag_class: 'source'
  system: 'Bifrost'
  country: 'Brazil'
  schedule: '*/5 * * * *'
  datasets:
    - name: 'IRIS_GENERIC_CLIENT'
    	domain: 'Generic'
      entity: 'Client'
      task_owner: 'Thauany Rocha - BRPGM0063@ambev.com'
      owner_team: 'Finance'
      active: True
   	 max_message_count: 40000
  		batch_size_to_persist: 8000
   	 max_lock_renewal_duration: 120
      execution_timeout: 360
      connection_id: 'servicebus_bifrost'
      metadata: !include ../metadata/generic_client.yaml
      notifier:
      	type: 'ms_teams'
        receivers:
        	- 'SEU_TOKEN_DO_TEAMS_PARA_RECEBER_MSGS'
      
~~~

## Estratégia de inserção

É importante ``frisar`` que como o **Service Bus** persiste as mensagens da fila conforme os eventos são sendo gerados nos sistemas origens, a estratégias de inserção utilizada para **pipelines** são sempre de utilizar o modo <kbd>append</kbd>, ou seja, todo dado que é consumido das filas são appendados nos arquivos ja existentes, mantendo a **imutabilidade** no dados, sem a necessidade de se utilizar estratégias mais complexas de inserção.

Também ressaltamos que o uso de ``overwrite`` para conectores de mensageira não é recomendável, pois a cada nova interação com a fila do conector de mensageira, a ação de sobrescrever iria substituir os dados atuais no data lake. Portanto, é sempre importante pensar em um caso de uso no qual isso seja benéfico para a sua regra de negócio. Como as mensageiras operam quase em tempo real, não seria correto sobrescrever os dados. Assim, no contexto de uso de mensageiras, a utilização de overwrite não é recomendada.

# Configurações Essenciais - Fila Servicebus

Atualmente, por exigência, é necessário configurar um parâmetro na fila do Service Bus, pois essa configuração permite que trabalhemos de maneira mais eficiente com nossa arquitetura. A configuração específica que devemos ajustar é a "**Message lock duration**", estabelecendo-a em **5 minutos**. 

Para todas as filas que planejamos utilizar, é crucial realizar essa alteração no "**Message lock duration**" para **5 minutos.** Valores inferiores a **5 minutos** podem resultar em falhas no seu pipeline.

Para efetuar essa mudança, você pode realizá-la no portal do Azure, diretamente na fila relevante, seja durante a criação da fila ou por meio do Service Bus Explorer. O essencial é garantir que a funcionalidade esteja habilitada para o período de **5 minutos**, evitando assim ocorrências inesperadas de erros.

***Imagem Exemplo:***

<img src="https://imgur.com/oNHJgPn.png" width="680" alt="Imgur">
