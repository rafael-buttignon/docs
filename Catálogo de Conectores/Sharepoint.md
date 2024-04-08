# Criação de pipelines para fontes Sharepoint

Nesta seção você encontra as instruções para a criação de uma DAG de extração de fontes de Sharepoint

## Como criar credenciais para o Sharepoint

Caso não possua credenciais de autenticação para o Sharepoint, siga o passo-a-passo informado pela própria documentação da Microsoft [**clicando aqui**](https://learn.microsoft.com/pt-br/sharepoint/dev/solution-guidance/security-apponly-azureacs#setting-up-an-app-only-principal-with-tenant-permissions).
**Apenas atente-se para que no campo Permission Request XML seja informado o seguinte valor:**
`<AppPermissionRequests AllowAppOnlyPolicy="true">  <AppPermissionRequest Scope="http://sharepoint/content/sitecollection/web" Right="FullControl" /></AppPermissionRequests>`

Desta forma será possível utilizar o Client Credentials como meio de autenticação do Sharepoint, garantindo à Iris peremissão suficiente para realizar as operações de leitura na página.

## Atributos das *tasks*
	
As ***tasks*** na ingestão de **Sharepoint**, assim como para outros protocolos de arquivos (como SMB e ABFS), visam varrer o diretório especificado e escrever os dados contidos no caminho.
Os parâmetros permitidos para esta DAG permitem utilizar o arquivo YAML no formato abaixo:

~~~yaml
datasets:
  - name:
    domain:
    entity:
    task_owner:
    owner_team:
    active:
    connection_id:
    file_type: 
    has_header: 
    file_path:
    column_separator:
    engine:
    file_pattern:
    encoding:
    list_name:
    list_filter_expression:
    sheet_name:
    backfill:
    ignore_trailing_white_space:
    ignore_leading_white_space:
    multiples_files:
    skip_rows:
    column_range:
    path_variables:
    notifier:
~~~
---
- ``name``: Trata-se do nome do processo, podendo também ser o nome do arquivo que está sendo utilizado na ingestão, visando facilitar a identificação.


  - ``Validação``: obrigatório, não nulo e string. 
---
- ``domain``: Dominio de negocio em que a fila esta representada.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``entity``: Entidade de negocio em que a fila esta representada.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``task_owner``: E a pessoa ``responsável`` que vai cuidar da task.

  - ``Validação``: obrigatório, não nulo e string.
---
- ``owner_team``: E o time do ``responsável`` pela task. 

  - ``Validação``: obrigatório, não nulo, string e CamelCase.
---

- ``active``: E para saber se sua task vai estar ``ativa`` ou ``não`` no caso ``habilitar`` e ``desabilitar`` a task.

  - ``Validação``: obrigatório, ``True`` ou ``False``
---
- ``connection_id``: O nome da ``connection`` criada na UI do ``airflow`` com as credenciais para a conexão com a fonte.

  - ``Validação``: obrigatório, não nulo, string e snake_case.
---
- ``file_type``:Atualmente é possível fazer a ingestão dos formatos csv, xlsx e xls. 
A declaração deve **sempre** ser da seguinte forma:
"csv", "xlsx","xls", "parquet", "json", "sharepoint_list" sem inclusão de simbolos ou letras maiúsculas.

  - ``Validação``: obrigatório, não nulo e string.
---
- ``file_path``:e o caminho de pastas ate o dado que se deseja consultar a partir da URL do Sharepoint, por exemplo "**Shared Documents**/TestingFolder/Test/test_blob_connection.csv".
  - ``Validação``: obrigatório, não nulo e string.
---
- ``has_header``:Se o cabeçalho esta ativo. 

  - ``Validação``: opcional, não nulo e booleano.
---
- ``column_separator``: E o delimitador que vamos usar para separar as colunas do dataset. 
	- ``Validação``: opcional, não nulo.
  - ``Exemplo``: ';' , '|'.
---
- ``engine``: Engine utilizada para a leitura de arquivos .xls ou .xlsx (via pandas). 
	- ``Validação``: opcional, não nulo.
  - ``Exemplo``: 'openpyxl'.
---
- ``file_pattern``: A regra para varredura dos arquivos no diretório, de acordo com um padrão Regex. Por padrão, ele varre arquivos com o valor informado no **file_type**.

  - ``Validação``: opcional, não nulo, string.
---
- ``encoding``: O processo de converter informações de um formato para outro, basicamente e a representação de dados em diferentes formatos.

	- ``Validação``: opcional, não nulo.
  - ``Exemplo``: utf-8. 
---
- ``list_name``: Caso os arquivos estejam em uma lista, informar o nome da lista

  - ``Validação``: opcional, não nulo, string.
---
- ``list_filter_expression``: Query a ser realizada na lista para filtrar os resultados/linhas desejadas

  - ``Validação``: opcional, não nulo, string.
---
- ``sheet_name``:No caso de ingestão de xls e xlsx, é neste campo que é informado de qual aba da planilha deve ser feita a ingestão.
  - ``Validação``: opcional, não nulo e string.
---
- ``ignore_trailing_white_space``: Quando ignore_trailing_white_space é definido como True, o Spark ignorará quaisquer espaços em branco no final de cada valor de campo. Por exemplo, se houver um campo no arquivo CSV com o valor **"Valor "**, com espaços em branco após o texto, definir ignore_trailing_white_space=True fará com que o Spark remova esses espaços em branco, lendo o valor como "Valor".
	- ``Validação``: opcional, não nulo. Aceito apenas ``True`` ou ``False``, surge efeito apenas no parse de arquivos CSV & TXT.
---
- ``ignore_leading_white_space``: Quando ignore_leading_white_space é definido como True, o Spark ignorará quaisquer espaços em branco no início de cada valor de campo. Por exemplo, se houver um campo no arquivo CSV ou TXT com o valor **" Valor"**, com espaços em branco antes do texto, definir ignore_leading_white_space=True fará com que o Spark remova esses espaços em branco, lendo o valor como "Valor".
	- ``Validação``: opcional, não nulo. Aceito apenas ``True`` ou ``False``, surge efeito apenas no parse de arquivos CSV & TXT.
---
- ``connection_id``: o nome da ``connection`` criada na UI do ``airflow`` com as credenciais para a conexão com a fonte.

  - ``Validação``: obrigatório, não nulo, string e snake_case.
---
- ``multiples_files``: Este parâmetro permite a ingestão de vários arquivos de uma pasta com o mesmo schema.
	- ``Validação``: obrigatório, não nulo. Aceito apenas ``True`` ou ``False``.
---
- ``path_variables``: Dicionário contendo todas as variáveis a serem customizadas pelos parâmetros **file_path** e **list_filter_expression**. Consulte como utilizar este parâmetro para tornar seu caminho dinamico [**clicando aqui**](https://iris.ambev.com.br/pt-br/iris-ingestion/deep-dive/dynamic-paths).  
	- ``Validação``: opcional, não nulo, dict.
---
- ``skip_rows``: **Utilizado em projetos SMB, Sharepoint e Azure Blob**, nesse parametro o usuário deve informar a quantidade de linhas que ele gostaria de pular do seu dataset. Exemplo: Caso o usuário opte por pular 3 linhas, ele começará a ingestão da 4ª linha.
- ``Validação``: opcional, não nulo. Aceito apenas numeros.
	-``Exemplo``: 1,2,3
---
- ``column_range``: **Utilizado em projetos Xslx e xls**, esse parametro permite o usuário selecionar as colunas que ele deseja realizar a leitura do dataset.(Lembrando que o contrato da ingestão tem que estar de acordo com o usecols selecionado pelo usuário.)
- ``Validação``: opcional, não nulo. Aceito apenas Letras em Caps seguido de numero.
	-``Exemplo``: "A1:G1"
---
- ``notifier``: É onde se configura as notificações para as ações na DAG.

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
  São os atributos **obrigatórios**, que todas dags devem ter:
~~~yaml
datasets:
  - name:
    active:
  	domain:
    entity:
    task_owner:
    owner_team:
    file_path: # ou list_name
    file_type:
    connection_id:
    notifier:
     
~~~
## Leitura de Arquivos via Shared Documents

A **leitura dos arquivos no Sharepoint** acontece via **Shared Documents** ou **Listas**. 

Para esta estratégia de Shared Documents, os dados são buscados diretamente na seção **Documents** do Sharepoint. A busca aos dados acontece de acordo com o caminho informado na variável **file_path**.
Lembrando que caso a leitura seja via Shared Documents, não é possível utilizar a variável **list_name**.
Em um exemplo de leitura de um arquivo .csv, os parâmetros relacionados ao Sharepoint seriam:

~~~yaml
	file_path: 'Shared Documents/TestingForWiki/Test/arquivo.csv'
  file_type: 'csv'
  multiples_files: False
~~~


#### Utilizando regex para varrer os arquivos

Caso a leitura seja de múltiplos arquivos, é possível filtrar o diretório final para que retorne somente os arquivos necessários:

~~~yaml
	file_path: 'Shared Documents/TestingForWiki/Test'
  file_type: 'csv'
  multiples_files: True
  file_pattern: '[a-zA-Z0-9]*.csv'
~~~

Por padrão, caso a variável **file_pattern** não seja informada, é retornada para buscar arquivos de acordo com o **file_type**.

## Leitura de arquivos via Listas

Para a estratégia de Listas, os dados são buscados diretamente na seção correspondente à lista contida do Sharepoint.
Lembrando que caso a leitura seja via Lists, não é possível utilizar a variável **file_path**.
Em um exemplo de leitura de um arquivo .csv, os parâmetros relacionados ao Sharepoint seriam:

~~~yaml
	list_name: 'nome_da_lista'
  file_type: 'csv'
  multiples_files: False
~~~

#### Utilizando regex para varrer os arquivos

Caso a leitura seja de múltiplos arquivos, é possível filtrar o nome dos arquivos (assim como para Shared Documents) para que retorne somente os arquivos necessários:

~~~yaml
	list_name: 'nome_da_lista'
  file_type: 'csv'
  multiples_files: True
  file_pattern: '[a-zA-Z0-9]*.csv'
~~~

Por padrão, caso a variável **file_pattern** não seja informada, é retornada para buscar arquivos de acordo com o **file_type**.


#### Utilizando queries para filtragem de registros

Também é possível filtrar o resultado da lista para que retorne somente os registros necessários para a consulta. Tomando o exemplo anterior, pode ser feito:

~~~yaml
	list_name: 'nome_da_lista'
  file_type: 'csv'
  multiples_files: True
  file_pattern: '[a-zA-Z0-9]*.csv'
  list_filter_expression: "Title eq 'Freddy'"
~~~

Com o parâmetro **list_filter_expression** no exemplo acima, somente os registros onde a coluna *Title* tiver o valor *Freddy* serão retornados.  Aqui estão alguns dos operadores mais comuns:

 - eq: Igual a (equal to)
 - ne: Diferente de (not equal to)
 - lt: Menor que (less than)
 - le: Menor ou igual a (less than or equal to)
 - gt: Maior que (greater than)
 - ge: Maior ou igual a (greater than or equal to)

Tanto o parâmetro **list_filter_expression** quanto o **file_path** permitem a utilização de [**Caminhos Dinamicos**](https://iris.ambev.com.br/pt-br/iris-ingestion/deep-dive/dynamic-paths). Sendo assim, poderia ser feito:

~~~yaml
	list_name: 'nome_da_lista'
  file_type: 'csv'
  multiples_files: True
  file_pattern: '[a-zA-Z0-9]*.csv'
  list_filter_expression: "Title eq 'Freddy' and Datetime gt :date_yesterday:"
  path_variables:
  	date_yesterday:
    	day_range: 1
      date_format: '%Y-%m-%d'
~~~

Desta forma, seriam retornados os registros cuja coluna *Title* tenha o valor *Freddy* e o campo *Datetime* seja maior que a data de ontem.



## Leitura de Itens de uma Lista

Também é possível que os itens de uma lista sejam ingeridos, como em uma tabela de um banco de dados. Para isso, informe o parâmetro **file_type** como **sharepoint_list**. Também é posssível utilizar o **list_filter_expression** para filtrar os resultados.

~~~yaml
	list_name: 'nome_da_lista'
  file_type: 'sharepoint_list'
  list_filter_expression: "Title eq 'Freddy' and Datetime gt :date_yesterday:"
  path_variables:
  	date_yesterday:
    	day_range: 1
      date_format: '%Y-%m-%d'
~~~

### Nomenclatura das colunas

Vale ressaltar que o nome das colunas que são exibidas na página da lista **podem não representar o nome padrão para a lista**. Isso acontece pois a nível de código, o Sharepoint estabelece alguns padrões no momento de criar o *nome interno* para a coluna.

Para consultar o nome interno da coluna, abra sua lista e vá em **List Settings**.
![sharepoint_lista_column1.png](/iris-ingestion/sharepoint_lista_column1.png)


Na seção **Columns**, clique na coluna que deseja encontrar o nome interno.
![sharepoint_lista_column2.png](/iris-ingestion/sharepoint_lista_column2.png)

Na barra de endereço, ao final da URL é apresentado *Field=(NOME DO CAMPO)*. Este valor é o nome interno a ser utilizado no metadata.
![sharepoint_lista_column3.png](/iris-ingestion/sharepoint_lista_column3.png)


Existem outras formas de consultar o nome interno das colunas das listas. [**Clique aqui**](https://www.sharepointdiary.com/2021/01/get-field-internal-name-in-sharepoint-online-using-powershell.html) e saiba mais.



