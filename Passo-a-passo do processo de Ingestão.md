# Criando uma DAG

O módulo source conta com a geração de **DAGs** **(Directed Acyclic Graph)** a partir de arquivos **yaml**. Estes, por sua vez, são criados e armazenados dentro do repositório de Ingestion no Github, denominado Correnteza (`correnteza/src/domain/source/[country]/[sistema de origem]....`). Cada arquivo forma uma **DAG**, e cada DAG deve contemplar todas as extrações de uma mesma origem e com um mesmo intervalo de acionamento (**schedule interval**). Cada **task** contempla uma fila do conector escolhido a ser consumida. Dessa forma, cada fonte conta com um conjunto de argumentos necessários para a construção das DAGs.

## Fluxo de criação de DAGs
	
  A criação de uma **DAG** ou **task** se torna bem fácil na arquitetura 2.0, pois ela é criada a partir de **argumentos** passados dentro de um **documento** do formato <kbd>**.yaml**</kbd>. Entretanto, é importante se atentar às informações contidas nestas documentações para evitar possíveis erros. 

Antes de começar a criação da sua **DAG**, certifique-se de que o dataset já está sendo ingerido. Caso negativo, entenda se já existe uma DAG pronta para este conector e que atenda ao domínio e banco a ser conectado. Se a resposta também for não, neste caso será indicado que seja criada uma nvoa DAG. Se já existir um fluxo com **tasks/datasets** neste escopo mas seu dataset ainda não existe nesta realidade, crie uma nova task. O fluxograma abaixo representa esta explicação.

<p align="center">
	<img width="80%" src="/iris-ingestion/dag_flow.png">
</p>

*Figura 1: Fluxo de criação de DAGs*

> **Atenção:** Sempre que começar o processo de **desenvolvimento** é importante ter certeza que seu ambiente virtual esteja **ativo**! Verifique se no terminal esta como **(.env)** caso não esteja é importante você entrar no ambiente virtual novamente a partir do comando: **source .env/bin/activate**
{.is-warning}

> **Atenção:** Caso não consiga realizar operações dentro do repositório Correnteza, uma das causas pode ser:
1- Gerar novamente o token de acesso ao Git, como previsto na documentação de pré requisitos
2- Certifique-se de que a sua VPN está ativa
3- Se os passos acima já foram feitos, execute o comando: **git config --global --unset https.proxy**
 {.is-warning}


## Gerar Templates

Para gerar seu template e criar os arquivos *yaml* mencionado anteriormente, é preciso ter acesso ao repositorio e ter concluído os pré requisitos do ambiente.

Após ter certeza que os passos citados acima estejam completos, você poderá iniciar o processo da criação de templates:

1. Para isso, basta executar o comando:

```
pyiris ingestion --create-template-files

```
2. Assim que você rodar esse comando ele irá listar todos os tipos de conectores desenvolvidos e você poderá escolher o conector desejado.

![connector_type.png](/connector_type.png)

3. Logo após escolher o conector ele retornará os argumentos necessários para você preencher o seu pipeline:

![information.png](/information.png)

## Atributos da DAG

A seguir constam os parâmetros da DAG, que visam gerenciar o job como um todo. É a partir deles que serão informados, por exemplo, o intervalo de execução de uma tarefa, país de origem do dado, sistema e país.

### Parâmetros obrigatórios

Alguns parâmetros já serão pré-setados. Entretanto, os abaixo devem ser informados manualmente:

- ``country``: o país onde fica alocado o time responsável pela DAG.

  - ``Validacão``: obrigatório, não nulo e string. Aceito apenas: "Argentina", "Bolivia", "Brazil", "Chile", "Colombia", "Ecuador", "Guyana", "Paraguay", "Peru", "Suriname", "Uruguay", "Venezuela" e "Saz".

---
- ``system``: o nome do systema onde será feita a **extração**.

  - ``Validação``: obrigatório, não nulo, string e snake_case.

---
- ``schedule``: esse é o intervalo de execução da DAG. Como já informado, uma DAG tem apenas um intervalo de execução e uma fonte de **extração**, contudo, pode conter várias **tabelas**.

  - ``Validação``: obrigatório, não nulo e string. Qualquer intervalo de execução aceito no airflow. 
  
### Parâmetros opcionais

- ``start_date``: esse atributo é responsável por indicar ao Airflow a partir de quando essa ``DAG`` será executada. Pode ser uma data **futura**, **presente** ou, até mesmo, **passada**. Para o módulo ``source`` é importante se atentar se alguma tabela dessa DAG optará por fazer um **backfill** (ingestão reatroativa). Caso nenhuma tabela utilize essa estratégia de ingestão, a ``DAG`` somente iniciará a execução em data presente ou futura. Por isso, mesmo que a *start_date* seja uma data passada, a ``DAG`` não fará nenhum tipo de execução retroativa. Caso haja alguma tabela dessa ``DAG``que opte por fazer um *backfill*, a *start_date* da tabela será o início da execução retroativa, desde que tal data esteja à frente da *start_date* da ``DAG``, que é também o marco inicial de execução de todas as tabelas. O conceito de *start_date*  deve ser interpretado junto ao  de *scheduler_interval*.

  - ``Validação``: não obrigatório e dict."
  - ``Exemplo``:
~~~yaml
  start_date:    
  	year: 2077    
    month: 12    
    day: 13    
    hour: 14    
    minute: 15
~~~
---
- ``dag_execution_timeout``: tempo máximo permitido para a execução desta instância de tarefa, se for além, aumentará e falhará.

  - ``Validação``: não obrigatório e integer, apenas ***"minutos"***: minimo 10, máximo 1440.
---

- ``retry_delay``: Quanto tempo após a dag falhar ele ira tentar novamente. 

  - ``Validação``: não obrigatório e integer, apenas ***"minutos"***: minimo 10, máximo 180.
---
- ``retries``: Quantidade de tentivas caso ocorra erro na task.

  - ``Validação``: não obrigatório e integer, minimo 0, máximo 9. 
---
- ``catchup``: Quando o parâmetro catchup de um DAG é definido como True, no momento em que o DAG é ativado no Airflow, o agendador iniciará uma execução de DAG para cada intervalo de dados que não foi executado entre os DAGs start_datee o intervalo de dados atual. Por exemplo, se nosso DAG estiver programado para ser executado diariamente e tiver uma start_dateexecução de 01/01/2021, e implantarmos esse DAG e o ativarmos em 01/02/2021, o Airflow agendará e iniciará todas as execuções diárias do DAG para Janeiro. O Catchup também será acionado se você desligar um DAG por um período de tempo e depois ligá-lo novamente.

  - ``Validação``: não obrigatório, boolean. Apenas False ou True.
---
- ``priority_weight``: prioridade por peso - [**Veja a Documentação de Como Utilizar**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/dag-pools) muito importante utilizar da melhor forma possível!

	- ``Validação``: Não obrigatório e integer, minimo 1,máximo 5.
---
- ``pool``: pool por execução - [**Veja a Documentação de Como Utilizar**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/dag-pools) muito importante utilizar da melhor forma possível!

	- ``Validação``: Não obrigatório, não nulo, string e snake_case.
---
- ``timezone``: local timezone. Por default utilizamos o timezone UTC, caso o schedule que você queira informar é o horário da sua região, você deve acrescentar o parâmetro timezone e informar a região, assim, na geração da DAG, nós vamos converter para o horário UTC.

	- ``Validação``: Não obrigatório, string.
  - ``Exemplo``:
  ```yaml
	dag:
		timezone: "America/Sao_Paulo"
		...
  ```
---

Após inserir todas as informações, os arquivos serão criados no caminho `correnteza/src/domain/source/[country]/[sistema de origem]....`


## Atributos da Task

Diferentemente dos parâmetros da DAG, os atributos de task visam especificar as propriedades de cada ingestão. Além dos parâmetros abaixo, é também necessário informar os atributos do conector em questão. É possível consultá-los no [Catálogo de Conectores](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/ingest#cat%C3%A1logo-de-conectores), seção mais abaixo.

### Parâmetros obrigatórios

- ``connection_id``: o nome da *connection* criada na UI do **airflow** com as credenciais para a conexão com a fonte.

  - ``Validação``: obrigatório, não nulo, string e snake_case.
---
- ``Dataset name``: é o nome da tabela. Ele deve ser **preenchido** exatamente da maneira como está no **banco** de dados, pois servirá como parte da **identificação** da tabela na *query* de consulta no banco de dados. Caso **contrário**, a *task* não funcionará.

  - ``Validação``: obrigatório, não nulo e string. 

---
- ``active``: é para saber se sua task vai estar **ativa** ou **não**, no caso **habilitar** e **desabilitar** a task.

  - ``Validação``: obrigatório, **True** ou **False**
---

- ``domain``: Dominio de negocio em que a fila esta representada.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``entity``: Entidade de negocio em que a fila esta representada.

  - ``Validação``: obrigatório, não nulo e string. 
---
- ``task_owner``: é a pessoa **responsável** que vai cuidar da *task*

  - ``Validação``: obrigatório, não nulo e string.
---
 
- ``owner_team``: é o time do **responsável** pela *task*. 

  - ``Validação``: obrigatório, não nulo, string e CamelCase.
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

### Parâmetros opcionais

- ``sync``: Indica os parâmetros que devem ser levados em consideração no momento de escrita. Consulte a documentação [aqui](https://iris.ambev.com.br/pt-br/data-transformation/explanation/syncservice)
  - ``Validação``: opcional, não nulo, dict. 
---
- ``depends-on``: Lista contendo as tasks que dependem dela para ser executada

  - ``Validação``: opcional, não nulo, list.

## Preenchendo o Metadata

### Como Funciona o Metadata

Todas as tasks, de todos os conectores existentes na arquitetura 2.0, contam com o atributo *metadata*. Trata-se de um ***yaml*** com algumas informações necessárias para a nossa arquitetura de metadados. Com eles conseguimos **documentar** os datasets, **alimentar** nosso catálogo de dados e ter controle de ***schemas*** e caminhos de pastas em nosso *data lake*.
  
Como dito acima, os **metadados** são inseridos como um atributo no ***yaml*** de pipeline, dentro de cada *task*. Contudo, por se tratar de um atributo com vários campos, ele deve ser inserido em outro *yaml* e apenas referenciado dentro da *task*. Por exemplo:
~~~yaml
datasets:
	metadata: !include ../metadata/example_metadata.yaml
~~~

> **Informação**: a *task* deve seguir o modelo de cada fonte, por exemplo: na fonte jdbc as *tasks* são tabelas, no *file system* são *files*, na *rest api* são *apis*...
{.is-info}


### **Organização do *yaml* do Metadata**
	
O ``yaml`` de metadados deve conter os seguintes campos.
  
---

- ``description``: a descrição do conteúdo da tabela; 
  	- ``Validação:`` Obrigatório, tipo string, detalhar o mais possível.
  
---

- ``dataexpert``: o nome e o e-mail da pessoa que tem conhecimento sobre os dados; 
  	- ``Validação:`` Obrigatório, tipo string.
      
---

- ``glossary``: Lista de termos da tabela presentes no glossário;
  	- ``Validação:`` Não obrigatório, tipo lista, detalhar o mais possível caso tenha.
      
---

- ``classifications``: classificação da tabela: PII ou não;
  	- ``Validação:`` Não obrigatório, tipo lista, detalhar o mais possível caso tenha.
	
Apenas os campos *description* e *dataexpert* são obrigatórios dentre os elencados acima. Os campos de *glossary* e *classifications* somente precisarão aparecer quando se enquadrarem no *dataset* em questão.
  
~~~yaml
description: 'This table contains informations of visit frequency for each active customer'
dataexpert: 'Rafael Augusto Buttignon - BRNEO524242@ambev.com.br'
glossary:
	- 'termo 1'
  - 'termo 2'
classifications:
	- 'PII'
~~~  
``schema:`` o schema do dataset.
  
O campo de *schema* também é obrigatório. Ele deve reportar todos os campos do *dataset* e, pra cada campo, deve conter: *name*, *description* e *type* como informações **obrigatórias**. Além disso, também é possível informar se se trata de PII ou se há alguma palavra do **glossário**.
~~~yaml
schema:
  - name: '_id'
    description: 'ID'
    type: 'string'
  - name: 'city_id'
    description: 'City ID'
    type: 'string'
  - name: 'radix'
    description: 'Radix'
    type: 'string'
  - name: 'state_abbreviation'
    description: 'State Abbreviation'
    type: 'string'
  - name: 'country'
    description: 'Country'
    type: 'string'
~~~   

### **Exemplos de Metadata**

Se alguma coluna do seu metadata for do tipo struct, você precisará passar as colunas edentadas.

``Exemplo:``
~~~yaml
schema:
  - name: '_id'
    description: 'ID'
    type: 'string'
  - name: 'week_day'
    description: 'Dia da Semana'
    type: 'struct'
    fields:
    	- name: 'description'
        description: 'Descrição do dia da semana da antecipação da visit'
        type: 'string'
      - name: 'key'
        description: 'valor da chave'
        type: 'integer'
  - name: 'country'
    description: 'Country'
    type: 'string'
~~~ 

Se caso você tiver uma coluna do tipo array, precisará passar um element_type e as colunas edentadas.

``Exemplo:``
~~~yaml
schema:
  - name: '_id'
    description: 'ID'
    type: 'string'
  - name: 'items'
    description: 'items'
    type: 'array'
    elementType:
    	type: 'struct'
    	fields:
        - name: 'discount'
          description: 'Desconto'
          type: 'string'
        - name: 'freeGood'
          description: 'Grátis'
          type: 'string'
        - name: 'price'
          description: 'Preço'
          type: 'string'
        - name: 'quantity'
          description: 'Quantidade'
          type: 'string'
  - name: 'country'
    description: 'Country'
    type: 'string'
~~~ 
	
Por último, mas não menos importante, o ***yaml*** do ***metadata*** deve ficar armazenado em uma pasta denominada *metadata*, que deve estar dentro da pasta com o nome do sistema, ou seja, no mesmo nível da pasta *pipeline*.
  
![Imgur](https://imgur.com/RM9YBwF.png)  

Por fim podemos ver um exemplo de um **metadata** que já está em **produção**. Caso ainda fique com dúvida pode verificar os **diversos** metadatas que estão já em **produção** é pegar como referência

![Imgur](https://imgur.com/GPf7l5n.png)

---

# Catálogo de Conectores
Na tabela a seguir você encontra nossos conectores prontos, assim como o roadmap para os que serão desenvolvidos. Caso haja uma mudança na estratégia e necessite uma repriorização, esta página será atualizada.

<table><thead><tr>
<th>Conector</th>
<th>Status</th>
<th>Data</th>
<th>Formato</th>
</tr></thead><tbody>
<tr>

<tr>
<td><a href="https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/connectors/jdbc" target="_blank" rel="noopener noreferrer">JDBC</a></td><td><b>Finalizado</b></td><td><b>Finalizado</b></td><td>SQLServer, PostGres, MySQL</td>
</tr>
<tr>
<td><a href="https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/connectors/rabbitmq" target="_blank" rel="noopener noreferrer">RabbitMQ</a></td><td><b>Finalizado</b></td><td><b>Finalizado</b></td><td>Filas</td>
</tr>
<tr>
<td><a href="https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/connectors/service-bus" target="_blank" rel="noopener noreferrer">ServiceBus</a></td><td><b>Finalizado</b></td><td><b>Finalizado</b></td><td>Filas</td>
</tr>
<tr>
<td><a href="https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/connectors/filesystem" target="_blank" rel="noopener noreferrer">File System (SMB)</a></td><td><b>Finalizado</b></td><td><b>Finalizado</b></td><td>Csv, Txt, Xls, Xlsx, Prx, Json</td>
</tr>
<td><a href="https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/connectors/blob" target="_blank" rel="noopener noreferrer">Blob (ABFS)</a></td><td><b>Finalizado</b></td><td><b>Finalizado</b></td><td>Csv, Txt, Xls, Xlsx, Parquet, Json, Delta</td>
</tr>
<tr>
  <td><a href="https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/connectors/rest-api" target="_blank" rel="noopener noreferrer">RestAPI</a></td><td><b>Finalizado</b></td><td><b>Finalizado</b></td><td>Autenticações Basic, OAuth2 e Customizadas</td>
</tr>
<tr>
<td><a href="https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/connectors/sharepoint" target="_blank" rel="noopener noreferrer">Sharepoint</a></td><td><b>Finalizado</b></td><td><b>Finalizado</b></td><td>Csv, Txt, Xls, Xlsx, Parquet, Json</td>
</tr>
<tr>
  <td><a href="https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/connectors/S3" target="_blank" rel="noopener noreferrer">S3 (AWS) </a></td><td><b>Finalizado</b></td><td><b>Finalizado</b></td><td>Parquet</td>
</tr>
<tr>
  <td><a href="https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/connectors/delta-share" target="_blank" rel="noopener noreferrer">Delta Share (Metastore) </a></td><td><b>Finalizado</b></td><td><b>Finalizado</b></td><td>Tabelas</td>
</tr>
<tr>
  <td><a href="https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/connectors/SFTP" target="_blank" rel="noopener noreferrer">Secure File Transfer Protocol (SFTP)</a></td><td><b>Finalizado</b></td><td><b>Finalizado</b></td><td>Csv</td>
</tr>
<tr>

  </table>


# Testando localmente

Para fazer o nossos teste localmente vamos precisar de de alguns passos simples.

Para prosseguirmos e necessário você ter o ambiente de trabalho configurado corretamente [**Instalação e Pré-Requisitos**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/access). Necessário fazer toda a instalação que está descrita no artigo a cima de instalação. 

Lembrando o intuito do teste localmente é para você ter uma **visão sistêmica** de como seu dado está. Para que você verifique, pipeline, conexão e metadata antes de subir para **produção**.

> **Observação:** É obrigatorio você fazer [**Instalação e Pré-Requisitos**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/access). Pois precisamos da instalação do JAVA. Para não ocorrer problemas, no momento do seu teste local!
{.is-warning}

 

## Iniciando o Teste Local

Na instalação do repositório quando você executa o comando **make dev-setup**, você faz instalação de algumas dependências, no nosso caso precisamos da **pyiris** que é com ela que vamos começar a execução do teste local.

>**Observação**: É obrigatório estar utilizando a versão do spark sincronizada com a pyiris! Caso você já possui o ambiente do Correnteza configurado, é esteja enfrentando problema, apague o ambiente virtual com o comando **rm -rf .env**, é execute o comando **make dev-setup** novamente!
{.is-info}

Para começarmos o teste local, o primeiro passa a se fazer é ligar nossa **VPN**, pois vamos precisar se conectar á uma rede para buscar dados, por tanto é bem provável que a rede que vamos se connectar esteja dentro do pull de redes da ambev. Então deixe sua **VPN ON**, sempre que for utilizar o teste local.

Agora que sua VPN, está conectada, vamos partir para pratica. Execute o comando abaixo em seu terminal, para habilitar o ambiente virtual.

```sh
source .env/bin/activate
```

Exportar Variavel de ambiente como TEST.

```sh
export ENVIRONMENT=TEST
```

Com o comando acima executado, você vai conseguir executar o comando abaixo sem problemas.

```sh
pyiris
```
Primeiro apenas execute o comando **pyiris** no seu terminal, para verificar se o modulo da **pyiris** já está instanciado com sucesso em seu projeto. Após receber o output, você pode executar o comando abaixo.

```sh
pyiris ingestion --run-test-local --file-name=NOME_DO_ARQUIVO_DA_DAG --task-name=NOME_DA_SUA_TASK
```

O comando acima é auto explicativo, você vai precisar passar o nome do arquivo da sua dag em **--file-name=**, é o nome da sua task em **--task-name=**

*Imagem Exemplo:*
![Imgur](https://imgur.com/vy9T3lY.png)

Como podemos ver, a imagem acima é bem simples de entender. Basta você mudar os parametros do comando é executar!

Assim que o comando acima for executado, você precisa passar as credenciais de acesso que você vai se conectar para obter as informações da fila, banco, ou rede especifica. Por isso é importante estar com a **VPN ON**

*Imagem Exemplo:*
![Imgur](https://imgur.com/i4KypfD.png)

O ponto importante para ressaltar, é a questão das informações extra de acesso. Cada conexão tem uma informação extra diferente da outra, mas basicamente é a mesma que você vai criar no Airflow, quando seu PR for para produção.

Para não ter diferenças, deixamos iguais o jeito que é montado local, com o jeito que você vai criar no Airflow de produção. Mas para não ter dúvidas, vamos deixar o padrão de toda as conexões abaixo.

***Rabbitmq***:
```sh
host: 'HOST'
login: 'LOGIN'
password: 'PASSWORD'
extra: {"vhost": "PASS_YOUR_VHOST"}
```

***ServiceBus***:
```sh
host: 'HOST'
login: 'LOGIN'
password: 'PASSWORD'
extra: {""}   #Não tem extra, Basta apertar enter!
```

***JDBC***:
```sh
host: 'HOST'
login: 'LOGIN'
password: 'PASSWORD'
extra: {"database_name": "PASS_YOUR_DB_NAME"}
```

> [**Para validar os dados do processo de ingestão ou encontrar os principais erros dos conectores, clique aqui.**](https://iris.ambev.com.br/pt-br/data-ingestion/arquitetura/2-0/databricks/how-to)
{.is-danger}


## Validando os dados gerados

Assim que você preencher suas credenciais e executar, a extração do sample de dados vai começar a rodar, dependendo do tamanho da fonte, pode demorar um tempo. Por isso, sempre é bom passar filtros é limitações de download de dados de cada fonte.

***Exemplo Filtro JDBC Task***:
```sh
filter: 'id < 5000'
```

***Exemplo Limitação Extração RabbitMQ Task***:
```sh
prefetch_count: 5000
```

***Exemplo Limitação Extração ServiceBus Task***:
```sh
max_message_count: 5000
```

Você pode aprender mais sobre esses parâmetros lendo a documentação de cada connector!

Portanto, assim que terminar a execução do comando, você vai ter um **output** de onde os dados da extração foi salvo.

*Imagem Exemplo:*
![Imgur](https://imgur.com/5d9VObl.png)

Podemos ver que no print acima, e passado o path de onde foi salvo a extração dos dados. No nosso caso precisamos ir até esse path, é copiar para que seja possível a visualização desses dados salvos.

*Imagem Exemplo:*
![Imgur](https://imgur.com/EdXZUMu.png)

Basta você ir no caminho que está descrito no seu output, no caso **test/local/data/rawzone** ou **test/local/data/trustedzone** é clicar com botão direito em cima da pasta de extração e selecionar a opção de **Copiar o Caminho**.

### Via Jupyter Notebook

Copiando o caminho, abra o arquivo ***data_exploration.ipynb*** para validar os dados que foram gerados na etapa anterior.

> **ATENÇÃO:** Você pode precisar instalar as extensões do Jupyter para o seu editor de texto. 
[**Extensão do Jupyter para Visual Studio Code**](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter).
[**Extensão do Jupyter para PyCharm**](https://www.jetbrains.com/help/pycharm/installing-uninstalling-and-upgrading-packages.html).
Também pode ser preciso configurar o ambiente virtual da Pyiris no Jupyter. [**Clique aqui**](https://code.visualstudio.com/docs/datascience/jupyter-notebooks) e saiba como. 
{.is-info}

![test_local_jupyter1.png](/iris-ingestion/test_local_jupyter1.png)

Preencha as informações das variáveis `country`, `system`, `domain` e `entity`, e execute todas as células do notebook. 

![test_local_jupyter2.png](/iris-ingestion/test_local_jupyter2.png)

> Este arquivo não precisa ser commitado junto com as suas mudanças. Por 
isso **não salve/commite** nenhuma alteração realizada no arquivo Jupyter.
{.is-warning}

### Via Terminal (PySpark)

Outra forma de validar seus dados é via terminal. Para isso, execute o seguinte comando.

```sh
pyspark                             
```

Executando o comando pyspark, ira iniciar uma instancia do pyspark em seu computador, para que seja possível fazer a análise do sample de dados.

Com o pyspark iniciado em seu terminal, execute o seguinte comando.

```sh
raw = spark.read.load(path='/DigiteSeuCaminhoCopiadoAqui')                          
```

*Imagem Exemplo:*
![Imgur](https://imgur.com/54G22hU.png)

Agora que você instanciou sua base de dados na variável raw, você pode começar a fazer consultas, para verificar se os dados estão corretos.

Alguns comandos exemplos:

```sh
raw.count() # a quantidade de dados

raw.show() # Os dados em forma de select

raw.limit(5).toPandas() # Os dados em forma de select mais bonito, passando um limit

raw.schema.names # Os nomes do schemas da sua tabela

raw.printSchema() # Schemas da sua tabela

raw.select(*).where("data_admissao like '2009-10%'") # Filtro Padrão com Where
```

*Imagem Exemplo:*
![Imgur](https://imgur.com/cPrIcon.png)

Para sair do modo interativo do pyspark no terminal use a tecla <kbd>CTRL + D</kbd> ou digite <kbd>exit()</kbd>.

> <a href="https://anheuserbuschinbev.sharepoint.com/:v:/s/irisplatform/ETDZOdDbuppIh625W_XWnvEBP-cBAB4bKvN0yZQ-NZn5gw?e=DSyqb5" target="_blank">**Assista a demontração:  Testando Dags Localmente**</a>
{.is-info}


Agora que você terminou seu teste localmente, e validou que a base de dados está correta, junto com seu pipeline, conexão e metadata, podemos prosseguir para a geração da DAG.


## Gerar Dag

Após realizar a validação dos dados, já é possível gerar a DAG. A geração de DAGs pode ser feita de duas formas:

### Automática

Para isso, é preciso adicionar todos os arquivos via git para serem commitados com os comandos abaixo:

```
git add .
git commit -m 'Mensagem de commit'
```

Desta forma, as DAGs já serão geradas e adicionadas ao seu commit, prontos para serem enviados para a sua branch remota.

### Via Pyiris CLI

Também é possível gerar a DAG através comando: 

```
pyiris ingestion --generate-dag --yaml-dag-file=source_system_rabbitmq_at_five_hours.yaml

```
>**Nota:**  Onde source_system_rabbitmq_at_five_hours.yaml será o nome do seu arquivo.
{.is-info}


Esse comando irá gerar um novo arquivo no diretório `src/application/dags/...` o qual terá o código fonte de sua DAG do Airflow.

Caso você queira gerar mais de uma DAG. Faça isso com o comando a seguir.

```
pyiris ingestion --generate-dag 

```

# Processo de Desenvolvimento

No Modulo Source, utilizamos o **gitflow** para qualquer contribuição no repositório do Correnteza. As branches são **sempre** baseadas na branch `main`, antiga `master`, e mergeados de volta na mesma branch `main`, que é a branch de ambiente de produção, conforme mostra a imagem abaixo:

![Imgur](https://cdn-images-1.medium.com/max/1600/1*iHPPa72N11sBI_JSDEGxEA.png)
 <center>Figura: Gitflow.</center>
 
Isso significa que você deve criar branches de `feature` ou ` bugfix` e integrá-las com a branch `main` solitando um *pull request* (PR). Leia o artigo [**aqui**](https://medium.com/@Web_Bailey/github-feature-branch-workflow-db3b24535d53) para obter mais informações sobre o processo de fluxo do Git.

## Criando a sua branch

Siga as intruções abaixo para criar e desenvolver na sua branch.
  
Faça checkout na branch main e puxe para a sua máquina as últimas alterações que foram feitas no repositorio:
  
```
git checkout main
git fetch -a
git pull origin main
```
  
Caso você prefira criar sua branch local com o git, utilize o seguinte comando:   
  
```bash
git checkout -b <username>/<feature/bugfix>-<branch-name>
```  

## **Nome da Branch**
Em primeiro lugar, o nome da sua branch deve ser **significativo**. Lembre-se de que esta é a primeira descrição de sua alteração de código e qualquer pessoa pode fazer checkout em sua branch para usá-la ou revisá-la.

<table>
  <thead>
    <tr>
      <th>Instância</th>
      <th>Branch</th>
      <th>Instrução</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Main</td>
      <td>main</td>
      <td>Deve ser mergeada com as branchs de Features/Fix</td>
    </tr>
    <tr>
      <td>Features/Bug Fix/Issue</td>
      <td>feat-* / fix-*</td>
      <td>Sempre criadas a partir da branch main. Feature que estou adicionando/expandindo ou bug que precisa ser corrigido. </td>
    </tr>  </tbody>
</table>

Observando o nome do branch, você pode entender do que se trata o desenvolvimento específico e sua finalidade.

Dê uma olhada nos exemplos abaixo:

> * `pedrosilva/fix-hash-transformation` - o desenvolvedor está tentando consertar o método de transformação de hash;
> * `joaosantos/feature-jdbc-segambev-table` - o desenvolver está subindo um novo dataset de uma tabela de um banco jdbc para ser integrado;
> * `mariapereira/hotfix-rabbitmq-credit-queue` - o desenvolvedor está tentando corrigir um bug no processo que lê os dados na fila de crédito de um RabbitMQ.

  
Caso você tenha criado sua *branch* diretamente na UI do Github, basta fazer o *checkout* para ela com o seguinte comando:
  
```bash
git checkout <username>/<feature/bugfix>-<branch-name>
```
   

O próximo passo é fazer um push das suas mudanças: 

```bash
git push -u origin <username>/<feature/bugfix>-<branch-name>
```
  

## Recado para quem está rodando o piloto*

>Atente-se quando for **atualizar** a branch com o comando **git checkout main/development**, pois existem dois tipos de branch:
 **Branch main** que é respectivamente **produção** 
 **Branch development** que é respecitivamente **desenvolvimento**. 
Então, caso você for **deployar** para ambiente de dev ou prod, **você deve escolher a branch certa** para que não tenha **divergencias de dados**, pois os repositorios podem estar com diferentes.
{.is-warning}


# Criando *Connections no Airflow*
	
Para cada conector existirá uma forma de criação de conexão no ambiente do Airflow. Para isso, escolha o tipo de conector que será utilizado:

<p></p>

- [**REST API**](/iris-ingestion/getting-started/connection/rest-api)

- [**JDBC**](/iris-ingestion/getting-started/connection/jdbc)

- [**RabbitMQ**](/iris-ingestion/getting-started/connection/rabbitmq)

- [**ServiceBus**](/iris-ingestion/getting-started/connection/servicebus)

- [**Blob (ABFS)**](/iris-ingestion/getting-started/connection/abfs)
	
- [**FileSystem (SMB)**](/iris-ingestion/getting-started/connection/smb)

- [**Sharepoint**](/iris-ingestion/getting-started/connection/sharepoint)

- [**S3**](https://iris.ambev.com.br/pt-br/iris-ingestion/getting-started/connection/S3)

# Deployment


É a implantação do serviço. Quando estamos falando em desenvolvimento de aplicações ou pipelines a palavra ganha o sentido de enviar atualizações e/ou mudanças de um ambiente para outro.

Ele marca as etapas do processo de construção ou mudanças de um software, por exemplo.

O termo deploy poder ser definido como um processo de implementação do código (html, css, Javascript, etc) do controle de origem ou artefatos de origem para uma plataforma de hospedagem. Ele também pode ser manual, parcialmente ou totalmente automatizado.


## Como analisar se meu deployment ocorreu com sucesso ?

No nosso caso, o deployment é acionado assim que você cria seu pull request, como você está utilizando a arquitetura 2.0, assim que você efetuar a criação do seu pull request no repositório do [**Correnteza**](https://github.com/cervejaria-ambev/correnteza), ira ser acionado uma verificação a nível de código, para saber se realmente está tudo OK!

Imagem exemplo:
![Imgur](https://imgur.com/02Q8uBN.png)

Por tanto, se o seu deploy falhar, você vai precisar analisar, acessando o link: [**Pipelines/Correnteza**](https://dev.azure.com/AMBEV-SA/IRIS-AnalyticsPlatform/_build?definitionId=10501)

Nessa área você vai ver todos os deployment que foram feitos. Então assim que seu pull request for criado você vai conseguir acompanhar ele por esse link. Caso aconteça algum erro com seu deployment, ele avisara com uma mensagem do qual erro **aconteceu**, para que você consiga resolver.



## Deploy da DAG no Airflow

O processo de deployment acontece automaticamente **após o merge** de seu PR.

Após o merge do PR, você pode entrar no ambiente do [Airflow](https://airflow-iris.ambev.com.br/home) de produção (ou [nonprod](https://airflow-iris.ambevdevs.com.br/home)) e verificar sua DAG.

![dag_airflow.jpg](/dag_airflow.jpg)


# Acompanhar a DAG

Para acompanhar e observar o funcionamento da nossa Dag, devemos acessar o site do [**Airflow Produção**](https://airflow-iris.ambev.com.br/login/?next=http%3A%2F%2Fairflow-iris.ambev.com.br%2Fhome), e entrar dentro da nossa Dag.

Dentro dela, conseguimos ver se a Dag conseguiu rodar com sucesso, quanto tempo levou, e diversas outras funcionalidades.

![Imgur](https://imgur.com/vKHRPv7.png)

Caso sua Dag, esteja com algum erro, clique na ``Dag/Task`` é selecione os ``Logs``, para poder apurar é resolver os possíveis problemas. 
	
![Imgur](https://imgur.com/meRWnuN.png)

Dentro de ``Log``, você vai conseguir acompanhar o fluxo de sua Dag, é também conseguir ir até o notebook aonde ela está sendo executada.

![airflow-log.png](/iris-ingestion/airflow-log.png)

Busque pela url iniciada com ``https://adb`` e clique no link.

![databricks-log.png](/iris-ingestion/databricks-log.png)

Na página que abrir, é possível encontrar o log da execução do job no Databricks.

## Validação dos dados

> [**Para validar os dados do processo de ingestão ou encontrar os principais erros dos conectores, clique aqui.**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/databricks)
{.is-danger}

# Deletar uma DAG

Caso seja preciso remover uma DAG do ambiente, o passo a ser seguido é:

1. Deletar o metadata da Dag que deseja excluir
2. Deletar o trecho da pipeline onde é gerado a DAG com o Metadata
3. Rodar novamente o comando make generate-dag
4. Subir sua alteração abrindo um PR
5. Informar o time Iris para que validem o PR e mergeiem