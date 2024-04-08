# Criação de pipelines para fontes BLOB (ABFS)

Nesta seção você encontra as instruções para a criação de uma DAG de extração de fontes de armazenamento blob azure. 
  
  
## Integração
 
 O acesso a leitura de arquivos em blob e feito através da biblioteca [**python azure-storage-blob**](https://learn.microsoft.com/en-us/python/api/overview/azure/storage-blob-readme?view=azure-python). Utilizamos os métodos principais dessa biblioteca para, a partir do formato de autenticação especificado pelo usuário, coletar os dados de formato CSV, TXT, PARQUET, DELTA, JSON, XLSX e XLS e converte-los em tabela no datalake.  


## Atributos das *tasks*
	
As ***tasks***, na ingestão de **blob** correspondem a um caminho ou path, cada task correspondem ao arquivo ou aos arquivos contidos naquele caminho  

Os atributos das tabelas, diferentemente dos atributos da **DAG**, podem variar de acordo com a estratégia pretendida para a **ingestão** de cada tabela. Por isso, é necessário se atentar à composição dos atributos **preenchidos** para cada tabela.
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
    protocol_type:
    column_separator:
    has_header:
    encoding:
    new_line:
    multiples_files:
    ignore_trailing_white_space:
    ignore_leading_white_space:
    sheet_name:
    storage_account_name:
    container_name:
    auth_type:
    skip_rows:
    column_range:
    connection_id:
    notifier:
~~~
---
- ``name``: é o nome da tabela. Ele deve ser ``preenchido`` exatamente da maneira como está no ``banco`` de dados, pois servirá como parte da ``identificação`` da tabela na ``query`` de consulta no banco de dados. Caso contrário, a task não funcionará.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``domain``: Dominio de negocio em que seu dado esta representado.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``entity``: Entidade de negocio em seu dado esta representada.

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
- ``file_path``:é o caminho de pastas ate o dado que se deseja consultar a partir do container, por exemplo "Brazil/Azure/CostManagement/teste/test_blob_connection.csv".
  - ``Validação``: obrigatório, não nulo e string.
---
- ``path_variables``: dicionário contendo todas as variáveis a serem customizadas pelo caminho dinâmico. Consulte como utilizar este parâmetro para tornar seu caminho dinamico [**clicando aqui**](https://iris.ambev.com.br/pt-br/iris-ingestion/deep-dive/dynamic-paths).  
	- ``Validação``: opcional, não nulo, dict.
---
- ``file_type``:Atualmente é possível fazer a ingestão dos formatos txt, csv, xlsx, xls, parquet, delta e json. 
Aqui a declaração deve **sempre** ser da seguinte forma:
"csv", "xlsx","xls", "parquet","delta", "json" sem inclusão de simbolos ou letras maiusculas.

  - ``Validação``: obrigatório, não nulo e string.
  
---
- ``protocol_type``:É o tipo de protocolo referente ao storage utilizado. 

  - ``Validação``: obrigatório para casos onde o file_type for "parquet", "json", "csv" e "txt" não nulo e string. Aceitamos apenas os tipos ``"abfss"`` e ``"wasbs"`` 
  
  
> Se o seu tipo de conta de armazenamento for **Armazenamento de Blob** você escolherá a opção **wasbs**.
Se for **Data LakeStorage Gen2** você escolherá **abfss**.
Caso você tenha alguma dúvida de qual é o tipo que você está utilizando consulte a [**Documentação da Microsoft**](https://learn.microsoft.com/pt-br/azure/storage/common/storage-account-overview) onde consta os tipos de Storage.
{.is-info}
---
- ``column_separator``: é o delimitador que vamos usar para separar as colunas do dataset. 
	- ``Validação``: obrigatório, não nulo.
  - ``Exemplo``: ';' , '|'.
---
- ``has_header``:Se o cabeçalho esta ativo. 

  - ``Validação``: obrigatório, não nulo e booleano.
---
- ``encoding``: Encoding é tornar dados legíveis de uma forma específica, como transformar letras em códigos. 
	- ``Validação``: obrigatório, não nulo.
  - ``Exemplo``: utf-8. 
---
- ``new_line:``: como será identificada uma quebra de linha.
	- ``Validação``: opcional, não nulo, String. Por padrao é **\n**.
---
- ``sheet_name``:No caso de ingestão de xls e xlsx, é neste campo que é informado de qual aba da planilha deve ser feita a ingestão.
  - ``Validação``: obrigatório, não nulo e string.
---
- ``storage_account_name``:conta de armazenamento onde o blob se encontra. 

  - ``Validação``: obrigatório, não nulo e string.
---
- ``container_name``:existem diversos containers dentro de uma conta de armazenamento, aqui se preenche o o container onde se encontra os dados de interesse . 

  - ``Validação``: obrigatório, não nulo e string.
---
- ``auth_type``: estamos trabalhando com três tipos de autenticação: access_key, sas_token, service_principal. Deve ser preenchido qual a forma de autenticação se deseja utilizar. 
Aqui a declaração deve **sempre** ser da seguinte forma:
"access_key", "sas_token","service_principal", sem retirada de simbolos ou letras maiusculas

Para mais informações sobre as formas de autenticação da azure consultar a seção [**Criando a(s) Connection(s) no Airflow**](/iris-ingestion/getting-started/connection)

  - ``Validação``: obrigatório, não nulo e string.
---
- ``ignore_trailing_white_space``: Quando ignore_trailing_white_space é definido como True, o Spark ignorará quaisquer espaços em branco no final de cada valor de campo. Por exemplo, se houver um campo no arquivo CSV com o valor **"Valor "**, com espaços em branco após o texto, definir ignore_trailing_white_space=True fará com que o Spark remova esses espaços em branco, lendo o valor como "Valor".
	- ``Validação``: opcional, não nulo. Aceito apenas ``True`` ou ``False``, surge efeito apenas no parse de arquivos CSV & TXT.
---
- ``ignore_leading_white_space``: Quando ignore_leading_white_space é definido como True, o Spark ignorará quaisquer espaços em branco no início de cada valor de campo. Por exemplo, se houver um campo no arquivo CSV ou TXT com o valor **" Valor"**, com espaços em branco antes do texto, definir ignore_leading_white_space=True fará com que o Spark remova esses espaços em branco, lendo o valor como "Valor".
	- ``Validação``: opcional, não nulo. Aceito apenas ``True`` ou ``False``, surge efeito apenas no parse de arquivos CSV & TXT.
---
- ``multiples_files``: Este parâmetro permite a ingestão de vários arquivos de uma pasta com o mesmo schema.
	- ``Validação``: opcional, não nulo. Aceito apenas ``True`` ou ``False``.
---
- ``skip_rows``: **utilizado em projetos SMB, Sharepoint e Azure Blob**, nesse parametro o usuário deve informar a quantidade de linhas que ele gostaria de pular do seu dataset. Exemplo: Caso o usuário opte por pular 3 linhas, ele começará a ingestão da 4ª linha.
- ``Validação``: opcional, não nulo. Aceito apenas numeros.
	-``Exemplo``: 1,2,3
---
- ``column_range``: **utilizado em projetos Xslx e Xls**, esse parametro permite o usuário selecionar as colunas que ele deseja realizar a leitura do dataset.(Lembrando que o contrato da ingestão tem que estar de acordo com o usecols selecionado pelo usuário.)
- ``Validação``: opcional, não nulo. Aceito apenas Letras em Caps seguido de numero.
	-``Exemplo``: "A1:G1"
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
Nesse exemplo, estamos especificando o caminho do arquivo desejado: ***FlyRNAi_data_baseline_vs_EGF.txt***. Essa abordagem permite a ingestão de um único arquivo específico. No entanto, é importante destacar que você pode tornar o caminho mais dinâmico ao utilizar parâmetros, permitindo maior flexibilidade na seleção dos arquivos a serem processados. [**Leia Sobre Caminho Dinâmico**](https://iris.ambev.com.br/pt-br/iris-ingestion/deep-dive/dynamic-paths).

---

2. Uma segunda abordagem é ler o último arquivo adicionado em um caminho específico. Nesse caso, a regra irá apenas recuperar o arquivo mais recente na pasta. Se houver 100 arquivos dentro desse caminho, ele selecionará o arquivo mais recentemente adicionado. Isso significa que, se um novo arquivo for adicionado nesse caminho todos os dias, ele sempre pegará o arquivo mais recente, ou seja, o "último" arquivo. - ``Exemplo``:

 ~~~yaml                         
 file_path: 'integration/consume_directory/'
 file_type: 'txt'
 multiples_files: False
~~~

Nessa abordagem, você irá recuperar o último arquivo adicionado no caminho **consume_directory/**. É importante deixar o caminho vazio após a pasta, pois o algoritmo irá selecionar automaticamente o arquivo mais recente encontrado nesse diretório. Ele buscará o último arquivo adicionado nesse diretório, independentemente do seu nome específico.

---

3. A terceira abordagem consiste em ler vários arquivos de uma pasta com o mesmo esquema. Nesse caso, é possível consumir um ou mais arquivos que possuem o mesmo esquema, bastando configurar da seguinte forma: - ``Exemplo``:

 ~~~yaml                         
 file_path: 'integration/consume_directory/'
 file_type: 'txt'
 multiples_files: True
~~~

Nessa abordagem para Blob, você irá consumir todos os arquivos. Você simplesmente fornecer o caminho da pasta para consumir todos os arquivos desejados. Por exemplo: **"integration/consume_directory/"**. Dessa forma, todos os arquivos dentro dessa pasta serão consumidos pelo sistema.

---
Esses foram alguns exemplos dos diferentes modos que você pode utilizar para consumir arquivos. No entanto, é importante lembrar que também temos o [**Caminho Dinâmico**](https://iris.ambev.com.br/pt-br/iris-ingestion/deep-dive/dynamic-paths), que torna a manipulação do caminho muito mais intuitiva e conveniente. Com o uso do [**Caminho Dinâmico**](https://iris.ambev.com.br/pt-br/iris-ingestion/deep-dive/dynamic-paths), você pode aplicar regras para simplificar é facilitar o trabalho com o caminho dos arquivos.

## Atributos obrigatórios
  São os atributos **obrigatórios**, que toda tabelas deve ter:
~~~yaml
datasets:
  - name:
    active:
  	domain:
    entity:
    task_owner:
    owner_team:
    storage_account_name:
    container_name:
    file_path:
    auth_type:
    file_type:
    sheet_name:
    has_header:
    connection_id:
    notifier:
     
~~~

## Atributos variáveis
**Backfill**
O *backfill* é a ingestão com execuções **retroativas**. Adotada essa estratégia, o airflow irá disparar diversas execuções da ``DAG``, a partir do *start_date* da tabela - desde que ele seja igual ou posterior ao da DAG, caso contrário, considerará o da ``DAG`` - somando o *scheduler_interval*, olhando para uma coluna de referência para fazer o filtro.
Ou seja, é possível fazer uma ingestão de uma tabela olhando para uma data do **passado** e ir **trazendo** os dados correspondentes àquele intervalo de execução, até chegar à data corrente.

**Exemplo:**
<p align="center">
	<img width="80%" src="https://imgur.com/bXDLUnH.png">
</p>

O backfill em abfs e possível de ser utilizado em situaçōes que os arquivos com dados de meses passados estão com nomes ou em pastas que denotam o mês aos quais os dados se referem, para acessar esta pasta ou arquivo é necessário utilizar a feature de [**Caminho Dinâmico**](https://iris.ambev.com.br/pt-br/iris-ingestion/deep-dive/dynamic-paths), com a variavel de data obrigatóriamente se chamando <kbd>date_backfill </kbd> .

É necessário tambem que o seguinte parâmetro da DAG <kbd>catchup = True </kbd> e o parâmetro no dataset <kbd>backfill = True </kbd>.

exemplo de arquivo yaml com feature de backfill 
~~~yaml
dag:
  dag_type: 'blob'
  dag_class: 'source'
 	system: 'iris'
  country: 'Brazil'
  schedule: '*/5 * * * *'
  catchup: True
  datasets:
    - name: 'abfs_test'
    	domain": 'TestDomain',        
      entity": 'TestEntity',
      active: True,
      task_owner: "Thauany Rocha - BRPGM0063@ambev.com",
      owner_team: 'Iris'
      backfill: True
      file_path: 'consume/data/:date_backfill:'
      path_variables:
      		date_backfill:
          	day_range: '0'
          	date_format: '%Y%m'
      file_type: 'csv'
      protocol_type: 'wasbs'
      multiples_files: False
      has_header: True
      column_separator: ','
      auth_type: 'sas_token'
      metadata: !include ../metadata/mes_global_tsample.yaml
      connection_id: 'abfs_task'
      notifier:
      	type: 'ms_teams'
        receivers:
          - 'SEU_TOKEN_DO_TEAMS_PARA_RECEBER_MSGS'
~~~


## Yaml Exemplo - BLOB

Exemplo, de como seu <kbd>.yaml</kbd> deve ficar:
~~~yaml
dag:
  dag_type: 'blob'
  dag_class: 'source'
 	system: 'iris'
  country: 'Brazil'
  schedule: '*/5 * * * *'
  datasets:
    - name: 'abfs_test'
    	domain": 'TestDomain'     
      entity": 'TestEntity'
      active: True
      task_owner: "Thauany Rocha - BRPGM0063@ambev.com"
      owner_team: 'Iris'
      file_path: 'consume/data/actors_awards.csv'
      file_type: 'csv'
      protocol_type: 'wasbs'
      multiples_files: False
      has_header: True
      column_separator: ','
      auth_type: 'sas_token'
      metadata: !include ../metadata/mes_global_tsample.yaml
      connection_id: 'abfs_task'
      notifier:
      	type: 'ms_teams'
        receivers:
          - 'SEU_TOKEN_DO_TEAMS_PARA_RECEBER_MSGS'
~~~