# Criação de pipelines para fontes - SFTP

Nesta seção você encontrara as instruções para a criação de uma DAG de extração de fontes como CSV.
 

## Integração

Para a integração do SFTP, usamos o metodo open_file da biblioteca [**Paramiko**](https://docs.paramiko.org/en/2.2/api/sftp.html). Utilizamos tabém algumas estratégias em pyspark, para a leitura de diferentes tipos de arquivos.

## Atributos das tasks

Na etapa de ingestão do **SFTP**, as tasks referem-se aos arquivos que serão ingeridos. Cada task representa um pipeline distinto.

Em outras palavras, cada SFTP pode estar conectado a vários conjuntos de dados (Datasets) para a obtenção dos dados. Em termos de uma DAG (SFTP), isso implica que múltiplas tarefas (Tasks) podem ser associadas a uma única DAG, exemplificando a versatilidade do processo de ingestão de dados.


~~~yaml
datasets:
    - name: 
      domain: 
      entity:
      task_owner:
      owner_team: 
      active: 
	  	file_path:
      path_variables:
      file_type:
      column_separator:
      has_header:
      encoding:
      debug:
      multiples_files:
      connection_id:
      notifier:
      
~~~
---
- ``name``: Trata-se do nome do processo, podendo também ser o nome do arquivo que está sendo utilizado na ingestão, visando facilitar a identificação.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``domain``: Dominio de negocio do processo.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``entity``: Entidade de negocio do processo.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``task_owner``: A pessoa ``responsável`` que vai dar manutenção na task.

  - ``Validação``: obrigatório, não nulo e string.
---
- ``owner_team``: O time do ``responsável`` pela task. 

  - ``Validação``: obrigatório, não nulo, string e CamelCase.
---

- ``active``: Para saber se sua task vai estar ``ativa`` ou ``não`` no caso ``habilitar`` e ``desabilitar`` a task.

  - ``Validação``: obrigatório, ``True`` ou ``False``
---
- ``file_path`` E o caminho, aonde o dado está armazenado.
	- ``Validação``: obrigatório, não nulo, string.
---
- ``path_variables``: dicionário contendo todas as variáveis a serem customizadas pelo caminho dinâmico. Consulte como utilizar este parâmetro para tornar seu caminho dinamico [**clicando aqui**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/dynamic-paths).    
	- ``Validação``: opcional, não nulo, dict.
---
- ``file_type``: O tipo de arquivo que vai ser consumido.
	- ``Validação``: obrigatório, não nulo, string, aceita apenas 'CSV'.
---
- ``column_separetor``: O delimitador que vamos usar para separar as colunas do dataset. 
	- ``Validação``: obrigatório, não nulo.
  - ``Exemplo``: ';' , '|'.
---
- ``has_header``: Se existe cabeçalho
	- ``Validação``: obrigatório, não nulo, string.
  - ``Exemplo``: 'true', 'false' ou 0.
---
- ``encoding``: O processo de converter informações de um formato para outro, basicamente e a representação de dados em diferentes formatos.
	- ``Validação``: obrigatório, não nulo.
  - ``Exemplo``: utf-8, ISO-8859-1. 
---
- ``multiples_files``: Este parâmetro permite a ingestão de vários arquivos de uma pasta com o mesmo schema.
	- ``Validação``: opcional, não nulo. Aceito apenas ``True`` ou ``False``.
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


## Tipos de Ingestão de Arquivos

![Imgur](https://imgur.com/z4WedWY.png)

Para trabalhar com esses diversos tipos, é necessário configurar adequadamente o caminho (path) conforme desejado. Aqui estão alguns exemplos práticos:

1. Uma maneira simples é acessar o arquivo desejado usando o caminho especifico - ``Exemplo``:   
 ~~~yaml                         
 file_path: 'integration/consume_directory/FlyRNAi_data_baseline_vs_EGF.txt'
 file_type: 'txt'
 multiples_files: False
~~~
Nesse exemplo, estamos especificando o caminho do arquivo desejado: ***FlyRNAi_data_baseline_vs_EGF.csv***. Essa abordagem permite a ingestão de um único arquivo específico. No entanto, é importante destacar que você pode tornar o caminho mais dinâmico ao utilizar parâmetros, permitindo maior flexibilidade na seleção dos arquivos a serem processados. [**Leia Sobre Caminho Dinâmico**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/dynamic-paths).

---

2. **Esta funcionalidade não está disponível no momento para o conector SFTP.**

---

3. A terceira abordagem consiste em ler vários arquivos de uma pasta com o mesmo esquema. Nesse caso, é possível consumir um ou mais arquivos que possuem o mesmo esquema, bastando configurar da seguinte forma: - ``Exemplo``:

 ~~~yaml                         
 file_path: 'integration/consume_directory/'
 file_type: 'csv'
 multiples_files: True
~~~

Nessa abordagem para SFTP, você irá consumir todos os arquivos que estiverem nesta pasta. Você pode simplesmente fornecer o caminho da pasta para consumir todos os arquivos desejados. Por exemplo: **"integration/consume_directory/"**. Dessa forma, todos os arquivos dentro dessa pasta serão consumidos pelo sistema.

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
      file_path:
      file_type:
      connection_id:
      notifier:
      
~~~

## Yaml Exemplo - SFTP de uma ingestão CSV

Exemplo, de como seu <kbd>.yaml</kbd> deve ficar:

~~~yaml
dag:
  dag_type: 'sftp'
  dag_class: 'source'
  schedule: '0 * * * *'
  system: 'Iris'
  country: 'Brazil'
  datasets:
    - name: 'UnidadeGeografica'
      active: True
      domain: 'SFTP'
      entity: 'Teste'
      task_owner: 'Jefferson Junio Lima Paixao'
      owner_team: 'Sales'
      file_path: '/BI/Processados/'
      file_type: 'csv'
      has_reader: True
      multiples_files: True
      column_separator: ','
      encoding: 'utf-8'
      connection_id: 'sftp_connection_hml'
      metadata: !include ../metadata/metadata_sftp_teste.yaml
      notifier:
        type: 'ms_teams'
        receivers:
          - 'SEU_TOKEN_DO_TEAMS_PARA_RECEBER_MSGS'

      
~~~
