# Criando Notebooks

Após realizar o login no [**Databricks**](https://adb-1450100671499314.14.azuredatabricks.net/?o=1450100671499314) será preciso criar um notebook para que possa validar seus dados. Para isso pedimos que você siga os passos abaixo.

 - Usar o usuário pessoal (pasta Users) para desenvolvimento de projetos e exploração de dados.

![img1.png](/img1.png)

- Se necessário,compartilhar com outras pessoas usar “permissions”.

![img2.png](/img2.png)

## Como ler os dados(ou partição)

1. Encontrando o meu dado no data lake.

O dado sempre se encontrará nos parâmetros que foram informados na task, como por ``exemplo``:
```
datasets:
	- name: 'Relatorio_Completo' 
	  active: True
	  domain: ‘Dominio’
	  entity: 'Entidade'

dbutils.fs.ls('/mnt/trustedzone/[Pais]/[Dominio]/[Entidade]')
```
``Exemplo:``

![img3.png](/img3.png)


2. Leitura do dataset

```
df = spark.read.load('/mnt/trustedzone/[Pais]/[Dominio]/[Entidade]')

display(df)
```

``Exemplo:``
![img4.png](/img4.png)

## Operações básicas com o Spark

3. Aplicando filtros
```
df = spark.read.load('/mnt/trustedzone/Brazil/Cora/Precos/PrecosProprios/')

display(df.filter(df.codigo_filial == 85).filter( df.codigo_cliente == 189))
```
``Exemplo:``
![img5.png](/img5.png)

## Escolha de cluster a ser usado

Após informar os parâmetros do seu dataset no notebook você precisará escolher o cluster que irá rodar esse teste. Para isso deixamos um cluster criado exclusivamente para cada torre. 

``dbw-cluster-nonprod-iris-torre-edanumber``

[**Neste link**.](https://adb-5055431857639398.18.azuredatabricks.net/login.html?o=5055431857639398#notebook/2868378029529432/command/2868378029529433) você encontrará todos os exemplos anteriores


# Erros mapeados

Alguns erros já forma mapeados durante o processo de Ingestion. Caso sua dúvida não esteja mapeada nos itens acima, pedimos que abra um issue no nosso [Board de Operações](https://dev.azure.com/AMBEV-SA/IRIS-AnalyticsPlatform/_boards/board/t/AnalyticsTeam/Customer%20Service), detalhando seu problema.

## Criação e Execução da DAG

### Erros Gerais

<details close>
<summary>Estou recebendo o erro de Message: PyIrisTaskException(ValueError('Could not parse datatype: int'))</summary>
Esse erro ocorre quando alguma coluna do seu metadata(Schema) está com o tipo errado. Por exemplo: a coluna recebe letras e você colocou como float ou integer.
</details>

<details close>
<summary>Estou recebendo o erro Message: PyIrisTaskException(AnalysisException())</summary>
Esse erro ocorre quando você está testando sua Dag localmente, mas esqueceu de apagar sua pasta de test/local/data. Isso acontece quando você faz um primeiro teste e dá certo a ingestão local, mas para os próximos testes é preciso alterar o Path de destino dessa DAG ou apagar sua pasta **data** para poder ingerir novamente seus dados nessa pasta.
</details>

</details>
<details close>
<summary>Quando eu tento instanciar a pyiris ou rodar o comando para testar minha Dag localmente, eu recebo erro de pyiris: command not found</summary>
Esse erro ocorre por que você está fora do seu ambiente virtual (.env). Para continuar seu desenvolvimento basta você digitar o comando **source .env/bin/activate** .
</details>

</details>
<details close>
<summary>Message: PyIrisTaskException(ValueError("Failed to connect to '172.1.1.1': timed out"))</summary>
Esse erro ocorre quando você está tentando executar o teste local mas esta com a VPN desligada, logo você não consegue alcançar o host para testar sua ingestão.
</details>


## SMB

<details close>
<summary>Message: PyIrisTaskException(SMBOSError(2, 'No such file or directory'))</summary>
Esse erro ocorre quando o tipo do do seu arquivo está errado, exemplo: o arquivo é um CSV e você colocou XLSX no campo file_type. 
</details>

<details close>
<summary>Message: PyIrisTaskException(IndexError('list index out of range'))]</summary>
Esse erro ocorre quando o caminho em que foi passado para buscar o(s) arquivo(s) está vazio. Certifique-se de que o caminho passado está correto e que possuem dados contidos nele.
</details>

## JDBC

<details close>
<summary>ReaderOptionsDatabaseException</summary>
Essa exceção ocorre quando os parâmetros de tipo de banco de dados não são suportados
</details>

<details close>
<summary>JdbcReaderException</summary>
Esse erro ocorre quando não conseguimos ler as mensagem de um JDBC.
</details>

## ServiceBus

<details close>
<summary>"Error when try to complete your messages, verify your dead letter queue messages, Error: {message}"
</summary>
Esse erro ocorre quando não consegue se consumir toda a mensagem...
</details>

<details close>
<summary>"Error when try to return dataframe! There was a problem with your message, check your queued messages, Error: {message}"
</summary>
Esse erro ocorre quando não conseguimos retornar um dataframe, verifique se há algum problema com suas mensagens a Fila
</details>

## RabbitMQ

<details close>
<summary>Estou tendo esse erro "Reached inactivity timeout at..."
</summary>
Esse ocorre quando o consumidor RabbitMQ atingiu o tempo limite de inatividade
</details>

<details close>
<summary>Estou recebendo o erro "Unable to acknowledge messages. Invalid or closed channel, or no messages to acknowledge."
</summary>
Esse erro ocorre quando não é possível reconhecer as mensagens... pode ser por conta do canal estar fechado ou inválido, ou então não há nenhuma mensagem para confirmar.
</details>

<details close>
<summary>No messages found in the queue
</summary>
Essa exceção acontece quando não foi encontrada mensagem na fila.
</details>

<details close>
<summary>Estou recebendo esse erro Reached execution timeout at...
</summary>
Isso acontece quando o consumidor RabbitMQ atingiu o tempo limite de execução
</details>

## Blob

<details close>
<summary>AbfsNotSupportedAuthType</summary>
Esse erro ocorre quando o tipo de arquivo ou autorização, não esta correto, isso ocorre quando você seleciona por exempo o tipo de autenticação sas_token, mas adiciona a chave do Service Principal.
</details>
