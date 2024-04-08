# Criação de pipelines para fonte Delta Share

Nesta seção você encontrarar as instruções para a criação de uma **DAG** de extração de fonte **Delta Share (MetaStore)**

## Processo de compartilhamento de dados

## Integração
A integração com o ``Delta Share`` não precisa de nenhuma biblioteca externa! Utilizamos o proprio Databricks de Ingestion, para se comunicar com o Unity Catalog, é assim receber dados compartilhados para a ingestão via connector ``Delta Share`` 

![sharepoint_lista_column1.png](https://imgur.com/ehWhFOc.png)

## Recebimento

Possuímos uma documentação que explica como podemos solicitar ao time Global os dados para receber no workspace de ingestão.

- [**Recebimento dos Dados**](https://iris.ambev.com.br/pt-br/data-serving/how-to/unity).

## Atributos das *tasks*
	
As ``tasks``, na ingestão do ``Delta Share``, correspondem ao path da tabela do Delta Share á ser consumido. Cada ``task`` é uma tabela. Todas as tabelas que compõem uma DAG são do mesmo serviço de ``Delta Share`` e utilizam o mesmo ``intervalo de execução``.

Os atributos do **Delta Share**, é bem simples. No caso é o connector mais simples de utilizar na arquitetura 2.0.

~~~yaml
	datasets:
    - name: 
    	domain: 
      entity:
      task_owner:
      owner_team: 
      active: 
      catalog:
      schema:
      notifier:
      
~~~
---
- ``name``: Certifique-se de configurar esse parâmetro com o nome exato da **tabela** desejada para consumo. Preencha o parâmetro com o nome da sua **tabela**, exatamente como é. ``Exemplo``: 'customer'

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
- ``catalog``: Certifique-se de configurar esse parâmetro com o nome exato do **catalogo** desejada para consumo. Preencha o parâmetro com o nome do **catalogo**, exatamente como é. ``Exemplo``: 'clients' 

	- ``Validação``: obrigatório, não nulo, string.
---
- ``schema``: Certifique-se de configurar esse parâmetro com o nome exato do **schema** desejada para consumo. Preencha o parâmetro com o nome do **schema**, exatamente como é. ``Exemplo``: 'tpch'

	- ``Validação``: opcional, não nulo, string.
---
- ``connection_id``: No caso do connector de **DeltaShare**, não precisamos de utilizar uma connection_id, pois o nosso meio de conexão já é o Databricks. Mas por padrão quando criamos o pipeline, o pipeline ira incluir o valor da connection como **"default_deltashare_connection"**. Não precisa mudar, apenas deixe o padrão que foi gerado para você, como default!

	- ``Validação``: obrigatório, não nulo, string.
  
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
      catalog:
      schema:
      notifier:
      
~~~
 
 ## Yaml Exemplo - DeltaShare

Exemplo, de como seu <kbd>.yaml</kbd> deve ficar:

~~~yaml
dag:
  dag_type: 'deltashare'
  dag_class: 'source'
  schedule: '0 * * * *'
  system: 'Iris'
  country: 'Brazil'
  datasets:
    - name: 'customer'
      active: True
      domain: 'Customer'
      entity: 'Client'
      task_owner: 'Rafael Augusto Buttignon'
      owner_team: 'Iris'
      catalog: 'samples'
      schema: 'tpch'
      connection_id: 'default_deltashare_connection'
      metadata: !include ../metadata/Customer_client.yaml
      notifier:
        type: 'ms_teams'
        receivers:
          - 'SEU_TOKEN_DO_TEAMS_PARA_RECEBER_MSGS'
~~~

> **Importante**: O Connector **Delta Share**, é o único connector que **não** possui teste local, pois precisamos do Databricks para fazer a intermediação com o **Unity Catalog**. Por enquanto não há como testar em ambiente local!
{.is-warning}
