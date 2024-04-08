# Criação de pipelines para fontes RestAPI

Nesta seção você encontra as instruções para a criação de uma DAG de extração de fontes de APIs

## Atributos das *tasks*
	
As ***tasks***, na ingestão de **RestAPI** correspondem a requisições de método em endpoints. Cada ***task*** é responsável por chamar um endpoint. 
Atualmente, os métodos permitidos pelo conector são: **GET** e **POST**. Todos os demais métodos são bloqueados.
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
    method:
    base_url:
    ordinations:
    pagination:
    	page_parameter_name:
      page_size_parameter_name:
      total_pages_parameter_name:
      amount_per_page:
      start_page:
    timeout:
    body:
    headers:
    params:
    path_variables:
    multiple_data:
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
- ``connection_id``: o nome da ``connection`` criada na UI do ``airflow`` com as credenciais para a conexão com a fonte.

  - ``Validação``: obrigatório, não nulo, string e snake_case.
---
- ``method``: é o método a ser utilizado na chamada da requisição
  - ``Validação``: obrigatório, não nulo e string.
---
- ``base_url``: é a url da API a ser chamada para a requisição. Deve conter a versão a ser chamada, caso contrário será utilizada a versão mais atual, segundo esta documentação.
	- ``Validação``: obrigatório, não nulo e string.
---
- ``ordinations``: lista contendo as ordenações que devem ser feitas. Esta funcionalidade é exclusiva das **APIs CORPORATIVAS**.

  - ``Validação``: opcional, não nulo e lista.
  - ``Exemplo``: 
~~~yaml
	ordenations:
  	- 'date desc'
      'name asc'
~~~
---
- ``pagination``: responsável por sinalizar que a chamada será feita em páginas. Se este parâmetro for passado, o conector irá identificar como a API interpreta os parâmetros que indicam a paginação. Atente-se para o fato de que **acontecerá uma ingestão por página**, isto é, os dados serão extraídos da API e ingeridos na Consumer Zone e Trusted Zone página a página.
	- ``Validação``: opcional, não nulo, dict.
  - ``Itens``:
  		- ``pagination_type``: indica qual formato de paginação é usada na API (**ITERATION** ou **OFFSET**)
  		- ``page_parameter_name``: nome do parâmetro que indica a página 
  		- ``total_pages_parameter_name``: (opcional) nome do parâmetro que indica a quantidade total de itens
  		- ``start_page``: (opcional) página pela qual o conector irá iniciar a ingestão
      	- ``chunks``: (caso o **pagination_type** for **OFFSET**) indica a quantidade de itens retornados por página
      
  - ``Exemplo de Paginação por Iteração``:  
~~~yaml
    pagination:
      pagination_tye: iteration
    	page_parameter_name: '?page'
      total_pages_parameter_name: 'total_pages'
      start_page: 0
~~~

  - ``Exemplo de Paginação por Chunks``:  
~~~yaml
    pagination:
      pagination_tye: offset
      page_parameter_name: '?page'
      chunks: 500
~~~
---
- ``timeout``: tempo pelo qual a requisição será feita até subir um erro de timeout
  - ``Validação``: opcional, não nulo e int.
---
- ``body``: corpo da requisição a ser enviada na chamada
  - ``Validação``: opcional, não nulo e dictionary.
  - ``Exemplo``:  
~~~yaml
	body:
  	Content: 'json'
~~~
---
- ``headers``: cabeçalho da requisição
  - ``Validação``: opcional, não nulo e dictionary.
  - ``Exemplo``:  
~~~yaml
	body:
  	Content: 'json'
~~~
---
- ``params``: conterá os parâmetros a serem passados na requisição API
  - ``Validação``: obrigatório, não nulo e dictionary.
  - ``Exemplo``:  
~~~yaml
	params:
  	name: 'Joao'
  	age: 20
~~~
---
- ``path_variables``: dicionário contendo todas as variáveis a serem customizadas pelos parâmetros **body**, **headers** e **params**. Consulte como utilizar este parâmetro para tornar seu caminho dinamico [**clicando aqui**](https://iris.ambev.com.br/pt-br/iris-ingestion/deep-dive/dynamic-paths).  
	- ``Validação``: opcional, não nulo, dict.
---
- ``multiple_data``: para casos em que a resposta da API traz vários resultados, especifique o nome do campo que retorna os dados. Caso não saiba, marque como ``[auto]``

  - ``Validação``: opcional, não nulo e string.
  - ``Exemplo``: 
~~~yaml
	multiple_data: '[auto]'
~~~
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

## Estratégias de consumo de API externas

# Estratégia de API - Conexão com Anaplan
Essa estratégia foi desenvolvida para facilitar a consulta de dados utilizando a API do Anaplan. O funcionamento dela é bem simples e vai facilitar muito a vida dos engenheiros que precisam interagir com a plaforma!

## Funcionamento da API
Antes de falar sobre a estratégia, vale explicar como é o funcionamento geral da API do Anaplan (*isso pode facilitar debugs futuros*).

### First things first
Antes de tudo, você precisa de algumas informações básicas:
1. <b>Workspace ID</b> - a workspace que você fará as consultas. Chamaremos essa variável de: `workspace_id`
2. <b>Model Name</b> - o nome do modelo que sua view está. Chamaremos essa variável de: `model_name`
    >*Nota: É importante que você tenha as informações do modelo, não do aplicativo. São coisas diferentes dentro do Anaplan.*
3. <b>View ID</b> - O ID da view que você fará a ingestão. Chamaremos essa variável de: `view_id`

>Se você não tiver essas informações, você pode pedir para alguém do business te passar ou acessar diretamente a view do Anaplan que você irá extrair. Essas informações estarão na URL da página.

### Funcionamento da API
Toda conexão via API do Anaplan passa por esses estágios:
1. É realizada uma chamada `POST` no endpoint de credenciamento para que seja criado um token Bearer único para suas interações. Esse token expira depois de alguns minutos de inatividade.
2. A partir da informações de workspace, é realizada uma chamada `GET` para fazer a listagem de modelos disponíveis para aquele usuário.
3. Com os identificadores únicos do seu modelo e sua view, é realizado um `POST` para gerar seu número de requisição. Esse número é como uma "senha" dentro da plataforma do Anaplan, você vai usar ela para solicitar a extração dos dados da sua view.
   >*No back-end, quando um `request_id` é gerado para uma view, o Anaplan gera os arquivos fisicamente no seu *file system* e os disponibiliza para download. É importante lembrar que isso impacta o storage do workspace.*
4. Com o número da requisição gerado, nós precisamos aguardar até que os arquivos estejam disponívels para extração. Nós fazemos isso realizando novas chamadas `GET` no endpoint até que ele retorne o estado como "COMPLETE" e o número de páginas disponíveis.
5. Tendo os arquivos disponíveis no servidor, é só executar chamadas `GET` nos endpoints de paginação. Cada página retornará uma string CSV, sendo que só a primeira página terá o cabeçalho.
6. Depois de realizar toda a páginação, é necessário enviar uma chamada `DELETE` para que o servidor saiba que a extração foi concluída, exclua os dados do armazenamento e encerre seu número de requisição.


## Atributos da Task
Foram adicionados poucos atributos novos, para facilitar a construção a partir do processoa atual de API. o YAML ficará assim:

- `api_type`: tipo de API/estratégia que será utilizada na execução do pipeline. Para utilizar a estratégia, deve ser preenchido com "ANAPLAN"
  - `validação`: string, não nulo.
---
- `anaplan_parameters`: informações básicas necessárias para interagir com a API.
  - `validação`: dicionário, não nulo.
  - Schema:
    - `workspace_id`: identificação única da workspace.
    - `view_id`: identificação única da view.
    - `model_name`: nome do modelo que será ingerido.

## Pontos de atenção
1. ***Sobre o tipo de paginação***: A estratégia funciona com ambos os tipos de paginação, mas notamos uma performance **MUITO** maior quando utilizada com o `pagination_type: "offset"`. Como a PyIris faz a escrita de cada página individualmente, quando interagimos em BULK podemos escrever inúmeros CSVs em uma lista e fazer uma escrita única de N páginas.
   1. 1. A PyIris vai escrever um dataset por paginação e o tempo de escrita vai variar de acordo com as dimensões do dataset.
   2. Quando usamos o tipo de paginação como '`iteration`' cada requisição às páginas vai gerar um processo de escrita, enquanto usando '`offset`' a quantidade chunks definida gerará uma única escrita de *N* páginas processadas. 
   3. O downside é claro: se o chunk for muito grande, vamos gastar mais tempo iterando nas requisições da API. Porém, nos testes feitos, o tempo gasto para fazer 50 requisições na API foi muito menor do que o necessário para escrever usando o '`iteration`'.
2. ***Sobre a URL base***: Para flexibilizar a evolução da estratégia em relação a API do Anaplan, deixamos o `base_url` disponível para customização. O único detalhe é não ter uma `'/'` no fim do endpoint.
3. ***Schema***: Durante a leitura do dataset há um processo de reforço de schema. Isso quer dizer que: o schema que você colocar no arquivo do YAML vai ser fielmente usado, tanto em questão de ordem quanto tipagem. Então, toda vez que houve atualização do schema na fonte, é necessário atualizar o schema no arquivo de metadata, para que o dataset reflita nas zonas de scrita.
   1. É importante lembrar que: se a ordenação das colunas mudar, dependendo da estratégia de escrita, vai ser necessário reprocessar o dataframe.
   2. Se a nova coluna for adicionada no fim do dataframe, não há nenhuma ação além de atualizar os metadados e recriar a DAG.


## Recomendações
1. ***Horário de execução do pipeline:*** A performance das extrações é diretamente conectada ao workload do workspace. Então, se você rodar suas extrações durante o horário de trabalho do business, isso vai impactar tanto o tempo da sua extração, quanto a experiência do usuário.

2. ***Renomeie as colunas que possam ser chamadas de year, month e day:*** Caso o particionamento do dataset esteja no padrão (isto é, por ano/mês/dia), certifique-se de que no Anaplan não existam colunas com os nomes year, month e day. Caso existam, renomeie antes de criar o metadata.


# Autenticação

Atualmente o conector de API permite autenticação via Basic, OAuth2 e autenticações customizadas. A configuração destas informações devem ficar na conexao do Airflow.

## Basic 

Para este formato de autenticação é necessário ter em mãos as informações a seguir:
- Login
- Password

### Criando uma conexão no Airflow

Em [Admin > Connections](https://airflow-iris.ambev.com.br/connection/list/) clique no (+) para adicionar uma nova conexão. 
- Em **Connection Id**, informe o nome referência da sua conexão, que será usada no **connection_id** no yaml da DAG. Deve seguir o seguinte padrão: restapi_[nome_da_fonte]_[autenticacao].
- Em **Connection Type**, informe **HTTP**.
- Em **Login**, informe o nome do **usuário**.
- Em **Password**, informe a **senha**.
- Em **Extra**, informe:
~~~json
{"auth_type": "basic"}
~~~

*Os demais campos devem ficar em branco.*

Clique em **Save**.

![airflow_restapi_basic.png](/iris-ingestion/airflow_restapi_basic.png)

## OAuth2

Para este formato de autenticação é necessário ter em mãos as informações a seguir:
- Client ID
- Client Secret
- URL de geração do token
- Escopo (se tiver)
- Apim subscription key (se tiver)

### Transformando as informações em JSON
O primeiro passo é transformar todas as informações em um json no formato abaixo:

~~~json
{
  "auth_type": "oauth2",
  "client_id": "VALOR DO CLIENT ID",
  "token_url": "URL DE GERAÇAO DO TOKEN",
  "scope": "ESCOPO SE TIVER",
  "apim_subscription_key": "SUBSCRIPTION KEY SE TIVER"
}
~~~

### Criando uma conexão no Airflow

Em [Admin > Connections](https://airflow-iris.ambev.com.br/connection/list/) clique no (+) para adicionar uma nova conexão. 
- Em **Connection Id**, informe o nome referência da sua conexão, que será usada no **connection_id** no yaml da DAG. Deve seguir o seguinte padrão: restapi_[nome_da_fonte]_[autenticacao].
- Em **Connection Type**, informe **HTTP**.
- Em **Password**, informe o **Client Secret**.
- Em **Extra**, informe o json gerado no passo anterior.

*Os demais campos devem ficar em branco.*

Clique em **Save**.

![airflow_restapi_oauth2.png](/iris-ingestion/airflow_restapi_oauth2.png)

## Autenticação Customizada

Visto que existem APIs que utilizam autenticações diferentes de OAuth2, esta alternativa irá suprir os formatos estáticos, isto é, autenticações que não necessitam gerar tokens.

### Inclusão da chave no cabeçalho ou corpo da chamada

Para chamadas de API com método **GET**, é preciso incluir o parâmetro responsável por armazenar o token de autenticação no campo **headers** ainda no código YAML da sua DAG. Entretanto, deixe o valor como vazio.
Supondo que o campo de autenticação na API seja o **Authorization**, o headers teria o campo abaixo, no seguinte formato:

~~~yaml
headers:
	Authorization: ''

~~~

Para chamadas de API com o método **POST** o parâmetro fica, por padrão, no campo **body**. Ele irá seguir o mesmo formato e regra do exemplo anterior:

~~~yaml
body:
	Authorization: ''

~~~

### Transformando as informações em JSON

O primeiro passo é preparar o JSON que será informado com o token da sua request. Seguindo os exemplos acima, o JSON ficaria no formato abaixo:

~~~json
{"Authorization": "Bearer EXAMPLE1234567"}
~~~

### Criando uma conexão no Airflow

Em [Admin > Connections](https://airflow-iris.ambev.com.br/connection/list/) clique no (+) para adicionar uma nova conexão. 
- Em **Connection Id**, informe o nome referência da sua conexão, que será usada no **connection_id** no yaml da DAG. Deve seguir o seguinte padrão: restapi_[nome_da_fonte]_[autenticacao].
- Em **Connection Type**, informe **HTTP**.
- Em **Password**, informe o **JSON criado no passo anterior**.
- Em **Extra**, informe:
~~~json
{"auth_type": "custom"}
~~~

*Os demais campos devem ficar em branco.*

Clique em **Save**.

![airflow_restapi_custom.png](/iris-ingestion/airflow_restapi_custom.png)

## Sem Autenticação

Caso a API não utilize nenhuma autenticação, ainda é necessário criar uma conexão no Airflow.

Em [Admin > Connections](https://airflow-iris.ambev.com.br/connection/list/) clique no (+) para adicionar uma nova conexão. 
- Em **Connection Id**, informe o nome referência da sua conexão, que será usada no **connection_id** no yaml da DAG. Deve seguir o seguinte padrão: restapi_[nome_da_fonte]_[autenticacao].
- Em **Connection Type**, informe **HTTP**.
- Em **Extra**, informe:
~~~json
{"auth_type": "noauth"}
~~~

![airflow_restapi_noauth.png](/iris-ingestion/airflow_restapi_noauth.png)

## Paginação

Visto que cada API tem sua forma de trabalhar com paginação, a forma de tornar esta funcionalidade genérica foi solicitar ao usuário quais são estes nomes na API em questão. Desta forma, trabalhando com paginação é **obrigatório** que você tenha em mãos as informações abaixo:

### Caso seja Paginação por Iteração
- **pagination_type**: Nome do parâmetro que indica a quantidade de itens por página
- **page_parameter_name**: Nome do parâmetro que indica a página


**total_pages_parameter_name**: Opcionalmente, é possível informar o nome do parâmetro que indica a quantidade de itens totais na requisição
**start_page**: Opcionalmente, também é possível informar em qual página a ingestão se inicia.


**Exemplo:** Tomando como exemplo a API a seguir:
![api-pagination.png](/iris-ingestion/api-pagination.png)

Os parâmetros no YAML seriam:
~~~yaml
    pagination:	
    	pagination_type: iteration
    	page_parameter_name: '?page'
      total_pages_parameter_name: 'total_pages'
      start_page: 2
~~~

### Caso seja Paginação por Chunks/Offset
A paginação por chunks ou offsets diz respeito à APIs que iteram com base nos registros contidos na requisição. 

**Exemplo:** Tomando com exemplo uma API que retorna 10000 registros, e o chunk/offset seja de 5000, o conector irá executar a ingestão para duas páginas. 
- A primeira página retornará os registros de 0 a 5000
- A segunda página retornará os registros de 5000 a 10000


Os parâmetros no YAML seriam:
~~~yaml
    pagination:	
    	pagination_type: 'offset'
    	page_parameter_name: '?page'
      chunk: 5000
~~~

**start_page**: Opcionalmente, também é possível informar em qual página a ingestão se inicia.

## Filtros

Para utilizar filtros na chamada da API, é necessário utilizar o parâmetro **params** informando todos os itens que serão considerados na requisição.

### Exemplo

~~~yaml
    params:
    	limit: '1'
      offset: '0'
      latitude: '-16.648301'
      longitude: '-49.2967995'
      uf: 'GO'
~~~


## Ordenação

Para utilizar ordenações na chamada da API, é necessário utilizar o parâmetro **ordinations** informando todos os itens que serão considerados na requisição.

Tomando como base os parametros abaixo:
~~~yaml
		base_url: 'https://reqres.in/api/users'
    ordinations:
    	- 'id desc'
				'name asc'

~~~

A chamada na API será: ``https://reqres.in/api/users?_order=id desc, name asc``

# Path Variables - Customizando os parâmetros da requisição

Podem existir cenários onde na requisição seja informada, por exemplo, a data do dia atual. Esta informação pode ser necessária no headers, no body ou no params. Sendo assim, foi acoplada a funcionalidade de [**Caminhos Dinâmicos**](https://iris.ambev.com.br/pt-br/iris-ingestion/deep-dive/dynamic-paths) para o conector de API.

## Exemplo

Supondo que na requisiçao utilizando o método GET, seja necessário informar o campo **params** com o seguinte valor:

~~~json
{"date": "2023-07-23", "yesterday": "2023-07-22"}
~~~

Ou seja, um campo contendo o dia atual e um campo contendo o dia anterior.
Sendo assim, a configuração do yaml seria o seguinte:

~~~yaml
      method: GET
      base_url: 'URL DA REQUISIÇAO API'
      params:
        date: ':date_now:'
        yesterday: ':date_last:'
      path_variables:
        date_now:
          day_range: 0
          date_format: '%Y-%m-%d'
        date_last:
          day_range: 1
          date_format: '%Y-%m-%d'
~~~

Neste exemplo, os campos **date_now** e **date_last** irão tratar a data de acordo com o formato necessário, respeitando o range informado. Com um **day_range** de 0 e um **date_format** de *%Y-%m-%d*, será considerada a data atual, seguindo o formato de ***ANO-MES-DIA***.

# Múltiplos dados

Visto que existem APIs nas quais as requisições retornam mais de um item, é comum que estas informações venham em encapsuladas em um parâmetro. Para isso, foi implementado a variável **multiple_data**. Nela, é preciso informar como é o nome deste parâmetro vindo da API.

## Exemplo

Tomando como exemplo a API a seguir:

~~~json
{
  "data": [
    {
      "id": 1,
      "email": "george.bluth@reqres.in",
      "first_name": "George",
      "last_name": "Bluth",
      "avatar": "https://reqres.in/img/faces/1-image.jpg"
    },
    {
      "id": 2,
      "email": "janet.weaver@reqres.in",
      "first_name": "Janet",
      "last_name": "Weaver",
      "avatar": "https://reqres.in/img/faces/2-image.jpg"
    }
  ],
  "page": 1,
  "per_page": 2,
  "total_pages": 6,
  "support": {
    "url": "https://reqres.in/#support-heading",
    "text": "To keep ReqRes free, contributions towards server costs are appreciated!"
  }
}
~~~

É possivel notar que os dados, de fato, vêm encapsulados no parâmetro **data**. Sendo assim, a variável multiple_data, no arquivo YAML deve especificar este nome:

~~~yaml
multiple_data: 'data'
~~~

Caso não seja conhecido o nome do parâmetro que encapsula os dados da requisição, é possível informar o valor **[auto]**. Desta forma, o conector irá buscar pelo primeiro parâmetro no formato lista vindo da requisição. 

>É importante ressaltar que neste cenário, é **IMPRESCINDÍVEL** que a API retorne os dados de forma encapsulada.{.is-warning}
~~~yaml
multiple_data: '[auto]'
~~~

# Yaml Exemplo - Rest API

Exemplo de como o <kbd>.yaml</kbd> deve ficar:
~~~yaml
dag:
  dag_type: 'restapi'
  dag_class: 'source'
 	system: 'iris'
  country: 'Brazil'
  schedule: '*/5 * * * *'
  datasets:
    - name: 'api_test'
    	domain": 'TestDomain',        
      entity": 'TestEntity',
      active: True,
      task_owner: "Thauany Rocha - BRPGM0063@ambev.com",
      owner_team: 'Iris'
      connection_id:
      method: 'GET'
      base_url: 'https://reqres.in/api/'
      multiple_data: 'data'
      pagination:
      	pagination_type: 'iteration'
        page_parameter_name: '?page'
        start_page: 1
      timeout: 200
      metadata: !include ../metadata/reqres_api_tsample.yaml
      notifier:
      	type: 'ms_teams'
        receivers:
          - 'SEU_TOKEN_DO_TEAMS_PARA_RECEBER_MSGS'
~~~


