# Criação de pipelines para fontes FileSystem (SMB)
 
Nesta seção você encontrara as instruções para a criação de uma DAG de extração de fontes como CSV, TXT, JSON, XLS e XLSX no FileSystem.
 
## Integração
Para a integração do FileSystem, usamos o metodo open_file da biblioteca smbprotocol [**SmbProtocol**](https://github.com/jborean93/smbprotoco) para abrir o arquivo usando o protocolo SMB. Ja para XLS e XLSX usamos serialytics para ler o arquivo e transformar em um dataframe Spark e para CSV usamos direto Spark mesmo.

## Atributos das *tasks*
	
Na etapa de ingestão do **SMB**, as tasks referem-se aos arquivos que serão ingeridos. Cada task representa um pipeline distinto.

Em outras palavras, cada arquivo do SMB pode estar conectado a vários conjuntos de dados (Datasets) para a obtenção dos dados. Em termos de uma DAG (SMB), isso implica que múltiplas tarefas (Tasks) podem ser associadas a uma única DAG, exemplificando a versatilidade do processo de ingestão de dados.

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
      new_line:
      mode:
      add_unb:
      debug:
      multiples_files:
	    ignore_trailing_white_space:
      ignore_leading_white_space:
      skip_rows:
      column_range:
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
- ``task_owner``: E a pessoa ``responsável`` que vai cuidar da task.

  - ``Validação``: obrigatório, não nulo e string.
---
- ``owner_team``: E o time do ``responsável`` pela task. 

  - ``Validação``: obrigatório, não nulo, string e CamelCase.
---

- ``active``: Para saber se sua task vai estar ``ativa`` ou ``não`` no caso ``habilitar`` e ``desabilitar`` a task.

  - ``Validação``: obrigatório, ``True`` ou ``False``
---
- ``file_path`` E o caminho, aonde o dado está armazenado.
	- ``Validação``: obrigatório, não nulo, string.
---
- ``path_variables``: Dicionário contendo todas as variáveis a serem customizadas pelo caminho dinâmico. Consulte como utilizar este parâmetro para tornar seu caminho dinamico [**clicando aqui**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/dynamic-paths).    
	- ``Validação``: opcional, não nulo, dict.
---
- ``file_type``: O tipo de arquivo que vai ser consumido.
	- ``Validação``: obrigatório, não nulo, string, aceita apenas 'CSV' , 'JSON' , 'TXT', 'PRX', 'XLS' e 'XLSX'.
---
- ``column_separetor``: E o delimitador que vamos usar para separar as colunas do dataset. 
	- ``Validação``: obrigatório, não nulo.
  - ``Exemplo``: ';' , '|'.
---
- ``has_header``: Se o cabeçalho esta ativo. 
	- ``Validação``: obrigatório, não nulo, string.
  - ``Exemplo``: 'true' ou 'false'.
---
- ``encoding``: O processo de converter informações de um formato para outro, basicamente e a representação de dados em diferentes formatos.
	- ``Validação``: obrigatório, não nulo.
  - ``Exemplo``: utf-8. 
---
- ``new_line:``: Como será identificada uma quebra de linha.
	- ``Validação``: opcional, não nulo, String. Por padrao é **\n**.
---
- ``mode``: E tipo de ação que vamos fazer com esse dataset. 
	- ``Validação``: Nesse campo vamos precisar de atenção para os tipos de arquivo que vamos precisar.
  - ``Exemplo``: 'r' para arquivos CSV e 'rb' para arquivos XLS e XLSX.
---
- ``debug``: Habilita os logs em baixo nível.
	- ``Validação``: opcional, não nulo. Aceito apenas ``True`` ou ``False``.
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
- ``prx_file_pattern``: **Utilizado em projetos PRX**, varre o .zip em busca dos arquivos que seguem o regex informado neste parametro.
	- ``Validação``: opcional, não nulo. String. Por padrão é o nome da task.
---
- ``add_unb``: **Utilizado em projetos PRX**, adiciona colunas de acordo com o nome do arquivo
	- ``Validação``: opcional, não nulo. Aceito apenas ``True`` ou ``False``.
---
- ``skip_rows``: **Utilizado em projetos SMB, Sharepoint e Azure Blob**, nesse parametro o usuário deve informar a quantidade de linhas que ele gostaria de pular do seu dataset. Exemplo: Caso o usuário opte por pular 3 linhas, ele começará a ingestão da 4ª linha.
- ``Validação``: opcional, não nulo. Aceito apenas numeros.
	-``Exemplo``: 1,2,3
---
- ``column_range``: **Utilizado em projetos Xslx e xls**, esse parametro permite o usuário selecionar as colunas que ele deseja realizar a leitura do dataset.(Lembrando que o contrato da ingestão tem que estar de acordo com o usecols selecionado pelo usuário.)
- ``Validação``: opcional, não nulo. Aceito apenas Letras em Caps seguido de numero.
	-``Exemplo``: "A1:G1"
---
- ``connection_id``: O nome da ``connection`` criada na UI do ``airflow`` com as credenciais para a conexão com a fonte.

  - ``Validação``: obrigatório, não nulo, string e snake_case.
---
- ``notifier``: E onde você configura para notifica-lo sobre as ações na DAG.

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
Nesse exemplo, estamos especificando o caminho do arquivo desejado: ***FlyRNAi_data_baseline_vs_EGF.txt***. Essa abordagem permite a ingestão de um único arquivo específico. No entanto, é importante destacar que você pode tornar o caminho mais dinâmico ao utilizar parâmetros, permitindo maior flexibilidade na seleção dos arquivos a serem processados. [**Leia Sobre Caminho Dinâmico**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/dynamic-paths).

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
 file_path: 'integration/consume_directory/director*.txt'
 file_type: 'txt'
 multiples_files: True
~~~

Nessa abordagem para SMB, você irá consumir todos os arquivos que o algoritmo encontrar com nomes que comecem com **"director"** e estejam no formato **"txt"**. No entanto, não é necessário especificar o padrão completo **"director*.txt**". Você pode simplesmente fornecer o caminho da pasta para consumir todos os arquivos desejados. Por exemplo: **"integration/consume_directory/"**. Dessa forma, todos os arquivos dentro dessa pasta serão consumidos pelo sistema.

---
Esses foram alguns exemplos dos diferentes modos que você pode utilizar para consumir arquivos. No entanto, é importante lembrar que também temos o [**Caminho Dinâmico**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/dynamic-paths), que torna a manipulação do caminho muito mais intuitiva e conveniente. Com o uso do [**Caminho Dinâmico**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/explanation/dynamic-paths), você pode aplicar regras para simplificar é facilitar o trabalho com o caminho dos arquivos.

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

## Yaml Exemplo - FileSystem de uma ingestão csv 

Exemplo, de como seu <kbd>.yaml</kbd> deve ficar:

~~~yaml
dag:
  dag_type: 'smb'
  dag_class: "source"
  system: 'Infocell'
  country: 'Brazil'
  schedule: '0 6 * * *'
  datasets:
    - name: 'vlc'
      domain: 'IrisTestDomain'
      entity: 'IrisTestEntity'
      active: True
      task_owner: 'Thauany Rocha - BRPGM0063@ambev.com'
      owner_team: 'Iris'
      file_path: 'cinfo\divulgacao\RPO\Maco_Margin_BEES\Bases\vlc.csv'
      file_type: 'csv'
      encoding: 'utf-8'
      mode: 'r'
      cleansing: False
      multiples_files: False
      column_separator: '|'
      has_header: True
      metadata: !include ../metadata/infocell_mm17.yaml
      connection_id: 'filesystem_infocell'
      notifier:
      	type: 'ms_teams'
        receivers:
        	- 'SEU_TOKEN_DO_TEAMS_PARA_RECEBER_MSGS'
      
~~~
 