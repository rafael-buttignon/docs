# Notificações

Atualmente, existem duas maneiras de ser receber notificações em caso de falha no seu pipeline: via **Datadog** ou **YAML**. Os alertas são enviados em um **canal do Teams**. Lembrando que os responsáveis pelo processo são o owner da DAG e o responsável do time.

## Fluxos de trabalho
![Imgur](https://imgur.com/0a8O8zR.png)


## Criando Webhook Teams
Na configuração do seu pipeline, você poderá definir o tipo serviço de notificações que você quer utilizar e quais vão ser os receptores das notificações. Segue abaixo como para criar e receber notificações no Microsoft Teams, que é o serviço que já está em funcionamento:

A primeira coisa a ser feita é criar um channel dentro do seu Team, como no exemplo:
![Imgur](https://imgur.com/1gEmdMQ.png)
Após isso, clique com botão direito no canal, é selecione a opção **Connectores**:
![Imgur](https://imgur.com/4UvSJnd.png)

---
Com o seu channel criado, selecione conectores e em seguida selecione para configurar o conector **Incoming Webhook**:
![Imgur](https://imgur.com/iNLOrT7.png)

---
Feito isso, preencha o nome do seu WebHook, seleciona uma imagem(se preferir), e clique em **Create**.
![Imgur](https://imgur.com/8WPtvpG.png)

---
Agora, para prosseguirmos para o próximo passo que é o Key Vault, é fundamental que você copie o seu WebHook. Por favor, certifique-se de armazená-lo em um local acessível. Isso é essencial porque essa informação será necessária posteriormente no processo.
![Imgur](https://imgur.com/v2I6uDh.png)

## Armazenando Webhook no Azure Key Vault

Para uma melhor compreensão, implementamos um processo automatizado via Forms que permite a criação simplificada de Key Vaults. Nessa automação, o recurso de criar seu próprio Webhook se torna uma tarefa fácil e rápida. [**Criação de Segredos no Azure Key Vault**](https://customervoice.microsoft.com/Pages/ResponsePage.aspx?id=GUvwznZ3lEq4mzdcd6j5NhjhQGZhSXxOjkoQeIwzDcdUQlNYV1FBNzgzMzBCRzUyMTU5MVNLVFpEWS4u)

Logo após acessar o link fornecido, você encontrará duas opções principais: "Criar novo segredo" ou "Atualizar segredo existente". A opção que você escolherá depende inteiramente do objetivo que você tem em mente.

Contudo, como o foco de nosso guia é auxiliar na criação de um novo segredo, nós recomendamos que você selecione a opção **"Criar novo segredo"**.


Agora que você optou por criar um novo segredo, o próximo passo requer que você preencha a sua **"Área"**, que basicamente se refere ao seu time.

Posteriormente, você encontrará o campo chamado **"Onde se conecta?"**, que requer a inserção do **"Sistema"** ou **"Contexto"** que corresponde ao seu pipeline. Este é o local onde você deve inserir as informações referentes aos dados preenchidos no seu pipeline.

Quanto à opção **"Tipo"**, é fundamental que você selecione **"Webhook"**. Isso é necessário para definir a natureza da informação que você está armazenando no Key Vault. 

Em relação à **opção 4**, esse é o espaço designado para que você insira o **link** do seu **Webhook**, que foi anteriormente criado no seu canal do **Microsoft Teams**.

Posteriormente, na **opção 5**, você terá a possibilidade de selecionar os **ambientes** nos quais deseja criar seu Webhook. Aqui, você tem a liberdade de escolher entre diferentes ambientes, como **produção** ou **desenvolvimento**.

***Imagem Exemplo:***
![Imgur](https://imgur.com/v53sjRb.png)

Conforme mencionado anteriormente, no **campo número 2**, é essencial preencher as informações de acordo com o **contexto** dos seus **dados**. Esta prática permite que você reutilize esse **Webhook** em outros produtos, como no *Correnteza*, *Transformation* e o *Intelligence*.

Neste campo, deve-se inserir as informações pertinentes ao seu "**Sistema**", "**Domínio**" e "**Entidade**", caso seja necessário. Esta nomenclatura tem como objetivo proporcionar um fácil reconhecimento e distinção do seu **Webhook** em relação a outros.

Vale lembrar que a **singularidade** do nome do Key Vault é essencial, uma vez que não é permitido ter dois Key Vaults com o mesmo nome. Portanto, caso seja necessário adaptar esse nome devido a conflitos ou por outras razões, a decisão final é sua. Esta flexibilidade garante que você possa personalizar o seu Key Vault de acordo com as suas necessidades específicas, para facil reutilização.

Após a criação do seu **Webhook**, você vai receber um **e-mail**, informando se o processo foi realizado com **sucesso** ou aconteceu algum **erro**.

***Imagem Exemplo:***
![Imgur](https://imgur.com/ZsLm0NG.png)

Com o Webhook no Key Vault criado, basta você copiar o nome que você deu em sua variavel é informar em seu pipeline! Basicamente como no exemplo, abaixo:

 ~~~yaml
       notifier:
        type: "ms_teams"
        receivers:
          - 'IrisDataGovernanceeWebhook'
~~~
---

## Ativando Notificações Teams

Pronto! Se por algum motivo o seu pipeline falhar, uma notificação chegaram no seu canal com os detalhes do erro e o link para acessar o pipeline do Airflow e debugar melhor o problema:
![Imgur](https://imgur.com/Cu5N1mY.png)

Também é fundamental que você ative as notificações do canal. Isso garante que você seja informado sempre que ocorrer um erro, permitindo que você atue rapidamente a qualquer situação que possa surgir. Para realizar essa ativação, basta seguir as instruções ilustradas na imagem que fornecemos.

![Imgur](https://imgur.com/8CuLE3b.png)

# Monitoramento 

Vamos aprender um pouco sobre monitoramento, e passar por cada tópico importante que precisamos saber sobre monitoramento e notificações.

## Padrão de Monitoramento que usamos ?

No caso nós adotamos um padrão especifico do Google, que seria os **Quatro Sinais Dourados**, que são uma série de métricas definidas pelo Google Site Reliability Engineering que são consideradas as mais importantes ao monitorar um sistema centrado no usuário:

- **Latência** : O tempo que leva para atender uma solicitação;
- **Tráfego** : Uma medida de quanta demanda está sendo colocada no sistema;
- **Erros** : A taxa de solicitações que falham;
- **Saturação** : Quão “cheio” é o nosso serviço, basicamente quão perto estamos de esgotar os recursos do sistema;

Entre essa metodologia existe outras diversas, como RED & USE, mas caso queria estudar mais sobre o assunto, vou deixar esse [**Artigo**](https://medium.com/thron-tech/how-we-implemented-red-and-use-metrics-for-monitoring-9a7db29382af), que aborda todas as metodologias, é assuntos de monitoramento.

## Por onde fazemos nosso monitoramento ?

Bom antes de começar explicar e mostrar os monitoramentos, precisamos saber por onde vamos fazer nossos monitoramentos, que seria pela ferramenta **DataDog**.

## Como Acessamos os Dashboards?
Existem **três principais monitoramentos** para a **Arquitetura 2.0**, sendo eles: **Airflow, Dags e RabbitMQ**.

**DashBoard Links:**

  - [**[Iris][Airflow]**](https://ab-inbev.datadoghq.com/dashboard/cm6-86t-a6s/irisairflow---new?from_ts=1657831736878&to_ts=1658436536878&live=true)
  
  O Dashboard **[Iris][Airflow]** é voltado para todos, mas com foco na equipe da **Iris**, que no caso esse Dashboard, inclui métricas gerais, que ajudam a precaver possíveis **incidentes** no ambiente.
  
  ---
  
  - [**[Iris][Airflow Dags]**](https://ab-inbev.datadoghq.com/dashboard/wtv-8mc-jny/irisairflow-dags---new?from_ts=1655844537212&to_ts=1658436537212&live=true)
  
  O Dashboard **[Iris][Airflow Dags]**, é voltado para o público que utiliza nossa arquitetura 2.0, pois nesse dashboard, você consegue verificar como está o funcionamento de sua **DAG/TASK**, monitorando a quantidade de falhas, duração e entre outros.
  
  ---
  
  - [**[Iris][PyIris - Metrics Airflow]**](https://ab-inbev.datadoghq.com/dashboard/qij-3w4-7gq/irispyiris-metrics?from_ts=1673355959239&to_ts=1673359559239&live=true)
  
  Com o Dashboard **[Iris][PyIris - Metrics Airflow]**, você obtém métricas especificas a nível de código, basicamente esse dashboard, vai lhe dar algumas informações, como: **Total Rows Trafficked, Total Elapsed Time, Total Execution Status** é diversas outras. 
  
  ---
  
   - [**[Iris][ServiceBus][Queues]**](https://ab-inbev.datadoghq.com/dashboard/mma-zh7-mti?refresh_mode=sliding&tpl_var_queue_name%5B0%5D=iris_update_market_allocation&view=spans&from_ts=1705407398565&to_ts=1705408298565&live=true)
  
  Com o Dashboard **[Iris][ServiceBus][Queues]**, você obtém métricas especificas importantes sobre o fluxo de suas filas! Podendo analisar se a entrada/sáida de msgs está ideal, para o seu cenário atual.
  
  ---
  
  - [**[Iris][Spark Measure]**](https://ab-inbev.datadoghq.com/dashboard/2hj-ssk-jg4/irisdatabricksspark-measure-20?refresh_mode=sliding&view=spans&from_ts=1704631110183&to_ts=1704717510183&live=true)
  
  Com o Dashboard **[Iris][Spark Measure]**, você obtém métricas especificas de Spark.
  
  ---
  
  - [**[Iris][RabbitMQ][Queues]**](https://ab-inbev.datadoghq.com/dashboard/axc-5g7-r2a?tpl_var_bifrost_environment=production&tpl_var_rabbitmq_queue=%2A&from_ts=1658422138754&to_ts=1658436538754&live=true)
  
  O Dashboard **[Iris][RabbitMQ][Queues]**, também é voltado para o público que utilizam nossa arquitetura 2.0, com esse dashboard você vai conseguir analisar se sua fila está necessariamente vindo do jeito que espera, como o total de mensagens, quantidade de mensagens, consumo de mensagens, entre outros...
  
  ---
  
  - [**[Iris][PostgreSQL]**](https://ab-inbev.datadoghq.com/dashboard/pt8-smu-84w?tpl_var_resource_group=ambev-iris-rg-eus-atcs-prod&tpl_var_resource_name=psql-prod-iris&from_ts=1658350343576&to_ts=1658436743576&live=true)
  
 O Dashboard **[Iris][PostgreSQL]**, é voltado para o time da **Iris**, com esse dashboard conseguimos avaliar como está nosso Banco de dados.
  
  ---
  
  - [**[Iris][Databricks]**](https://ab-inbev.datadoghq.com/dashboard/v3s-9mv-5uh?tpl_var_databricks_cluster_name=%2A&tpl_var_databricks_resource_name=%2A&from_ts=1657831947196&to_ts=1658436747196&live=true)  
  
  O Dashboard **[Iris][Databricks]**, é voltado tanto para o público, quanto para o time da Iris, com esse esses dashboards, conseguimos ter diversos insights para saber como está andando nosso ambiente do Databricks! Os gráficos do Spark, mostram os piores ofensores que estão afetando lentidão no Databricks.
  
  ---
  
  - [**[Iris][Databricks - In-Depth Detailing]**](https://ab-inbev.datadoghq.com/dashboard/7ew-4jg-z56?tpl_var_databricks_cluster_name=%2A&tpl_var_databricks_resource_name=%2A&from_ts=1658350347571&to_ts=1658436747571&live=true)
  
  O Dashboard **[Iris][Databricks - In-Depth Detailing]**, é o mesmo do **[Iris][Databricks]**, só que bem mais detalhado em algumas métricas, então esse Dashboard é feito para buscar Insights mais profundos é Debugar o ambiente que estiver com a saturação muito acima do normal.
  
  ---
  
  - [**[Iris][DataLake]**](https://ab-inbev.datadoghq.com/dashboard/vxz-2qq-uk7?tpl_var_storage_account=%2A&from_ts=1657832049575&to_ts=1658436849575&live=true)
  
  O Dashboard **[Iris][DataLake]**, é voltado tanto para o público, quanto para o time da Iris, com esse dashboard conseguimos analisar oque está acontecendo com nosso DataLake, principalmente se ele está recebendo muita requisição de **ERRO**!
  
  ---
  
  - [**[Iris][APIs]**](https://ab-inbev.datadoghq.com/dashboard/ym4-bex-zjr?from_ts=1658350450162&to_ts=1658436850162&live=true)
	
  O Dashboard **[Iris][APIs]**, é voltado para o time da Iris, com esse dashboard conseguiremos monitorar como está rodando nossas APIs.

---

No momento são esses Dashboards que temos **disponível**, mas com esses já conseguimos obter diversos insights de como está funcionando os ambientes! Também temos alertas, que vai ser abordado no final desse artigo.

## Como Monitoramos o Airflow ?

Como falado acima, essa documentação é completa para o monitoramento de todos os Dashboards, mas vamos abordar alguns dashs mais específicos para a Arquitetura 2.0. Mas o padrão segue para todos os dashboards! Vamos lá.

Primeiramente vamos analisar o nosso Dashboard [**[Iris][Airflow]**](https://ab-inbev.datadoghq.com/dashboard/cm6-86t-a6s/irisairflow---new?from_ts=1657831736878&to_ts=1658436536878&live=true), como falado anteriormente esse dashboard é totalmente voltado para compreendermos se nossa infraestrutura está tudo bem, para trabalharmos com proatividade!

Vamos entrar mais a fundo, para entender cada detalhe.

Assim que entramos no Dashboard, damos de cara com algumas variáveis de ambiente. Que no caso desse dashboard, seria as variáveis de *cluster_name* e *aks_namespace*. Lembrando que cada dashboard e diferente! Nessas variáveis você pode setar qual ambiente você quer analisar. Como vamos analisar o ambiente de produção, então já está tudo correto.

![Imgur](https://i.imgur.com/K6rC5Po.png)

---

A nossa primeira visão gráfica, iremos ver a saturação, no caso a saturação significa, ***"Quão “cheio” é o nosso serviço, basicamente quão perto estamos de esgotar os recursos do sistema"***. Nessa visão, você vai conseguir ver a média de quantos porcento A CPU, Memoria e Disco estão sendo usados. Também nos gráficos a baixos, você vai conseguir ter uma noção de comparação de como estava o uso de CPU, Memoria e Disco na semana anterior comparada com agora, esse tipo é muito interessante para saber se houve muito acréscimo de uso ou menos uso.

![Imgur](https://i.imgur.com/T5Tkl8l.png)

---

Já nessa segunda visão conseguimos visualizar o Scheduler Uptime, que está marcado de vermelho, no canto superior esquerdo. O Scheduler Uptime é basicamente o quanto de tempo que o nosso serviço pode ficar fora do ar em 1 ano. Pode ver que está definido como 96%, então em 1 ano, daria algumas horas que o sistema pode ficar fora do ar, o Scheduler Uptime, foi definido pensando em SLO's, então esse e a nossa porcentagem de tempo que podemos ficar offline.

Logo um pouco abaixo, também marcado de vermelho, temos o CPU Monitor Nodes, que no caso e o Monitoramento de CPU, nesse monitoramento ele vai mostrar, a porcentagem de tempo que a CPU passou em um estado ocioso, a porcentagem de tempo que a CPU gastou executando processos de espaço do usuário é também a porcentagem de tempo que a CPU gastou executando o kernel.

Ao lado também teremos o Memory Monitor Nodes, que também segue essa mesma linha do CPU, só que no caso da memoria é a Memoria Total, Memoria Usada, é Memoria Disponível

![Imgur](https://i.imgur.com/u0mseJd.png)

---

Agora em Tráfego nós falamos respectivamente ***uma medida de quanta demanda está sendo colocada no sistema***. Sabendo disso temos o Total de Tasks Rodada no período de tempo em que você visualizar. 

Temos também o Scheduler Tasks que faz a comparação entre o mês passado e agora, então nesse dashboard, conseguimos obter insights de como está o agendamento de tarefas, se ficou muita tarefa agendada pois não tinha slots disponível no pool, etc...

No gráfico mais para esquerda temos o Airflow Monitor Executor Tasks, que conseguimos visualizar como está o numero de slots disponível.

![Imgur](https://i.imgur.com/kiFVGj5.png)

---

No lado esquerdo superior, temos os Pods Available e Pods Desired, isso é bem interessante de visualizar, pois nossa infraestrutura contém 4 Pods, que são (2 Airflow Scheduler, 1 Airflow-WebServer, 1 Airflow-Statsd). Qual quer número abaixo de 4, com certeza estará com problemas na nossa infraestrutura.

Agora logo a baixo, temos a Latência, que é representada por o ***tempo que leva para atender uma solicitação***. A latência conta com diversos gráficos importantes.

O primeiro que temos é o Parse Import All Dags, que seria o tempo total para importar nossas dags, nesse gráfico conseguimos obter insights de como está nosso tempo estimado, para importar nossas dags, conseguimos também visualizar a comparação de tempo agora, versus mês passado. 

O último gráfico que temos é o Dag Task Duration AVG, que seria a média de duração das Tasks, que também faz a comparação de agora versus mês passado.

![Imgur](https://i.imgur.com/q24XEB8.png)

---

Por fim temos os Erros e Logs, que é representando por a ***taxa de solicitações que falham***.

Em erros, temos muitos insights que podem nós fazer descobrir a taxa de operadores que mais falham, é quais são os picos com mais falhas. O primeiro dashboard que temos é as tarefas com falhas, o dash faz a somatório de todas as falhas nesse intervalo de tempo.

Ao lado temos também, o gráfico de Tasks com falhas e sucessos, nesse gráfico conseguimos acompanhar o pico de sucesso e falha, se houve alguma quebra de padrões.

Logo a baixo, temos dois gráficos que retratam as falhas dos operadores, primeiro é o Operador do Databricks é o segundo é o Operador do Python. Nesses gráficos conseguimos verificar a taxa de sucesso e falha também.

Por fim temos os Logs do Airflow que contem erros. Nesses Logs conseguimos obter informações de logs, é informações que está acontecendo em tempo de execução dentro do AKS. Com esses Logs conseguimos descobrir se algum método ou aplicação está quebrando dentro do nosso Airflow, é assim concertar o mais rápido possível.

![Imgur](https://i.imgur.com/EEM5kQ2.png)

---

## Como Monitorar minha Dag Específica ?

Bom agora vamos monitorar as nossas Dags/Tasks, no exemplo acima aprendemos a como monitorar o Ambiente completo do Airflow, agora podemos entrar mais a fundo é obter insights específicos de cada DAG.

Para podermos analisar vamos entrar no Dashboard de [**[Iris][Airflow Dags]**](https://ab-inbev.datadoghq.com/dashboard/wtv-8mc-jny/irisairflow-dags---new?from_ts=1655844537212&to_ts=1658436537212&live=true), aqui podemos verificar alguns pontos mais específicos, vamos lá.

Entrando no Dashboard, conseguimos verificar algumas variáveis de ambiente, que podemos filtrar.

![Imgur](https://i.imgur.com/eGN2V0z.png)

---

A primeira visão que temos do dashboard é o tráfego, que está escrito a cima, qual quer dúvida volte para relembrar. 

Podemos analisar que temos a porcentagem de Sucesso em Dag, é também o total de tasks finalizadas, que são bons insights para comparar, se o rate estiver abaixo de uma quantia estipulada por você, com certeza é hora de rever oque está acontecendo com a DAG/TASK.

Também temos o Numero Total de tarefas ao longo do tempo, que faz a comparação de agora versus a semana passada. Conseguimos verificar se a quantidade de Tasks cresceu ou diminuiu ao longo desse período. 

![Imgur](https://i.imgur.com/oXHHk1h.png)

---

A segunda visão temos a Latência, o primeiro gráfico mostra o Rank de Dags/Tasks com média na duração em minutos, podemos verificar quais Dags/Tasks estão com maior média de duração, é assim verificar sé é uma anomalia ou não.

O segundo gráfico, nos mostra o Top 10 das piores Dags/Taks que mais demoram. Conseguimos verificar por esse gráfico qual foi a ultima Dag/Task que rodou é mais demorou, por fim tirar o insight sé realmente está normal esse tempo de duração, ou se precisa de alguma refatoração a nível de código.

![Imgur](https://i.imgur.com/OK4cBYE.png)

---

Para finalizar temos os Erros, que vai nos mostrar diversos indicadores, para metrificar nossas Dags/Tasks.

Começando pelo primeiro, temos a porcentagem de Falhas por Dag, podemos verificar se não houve uma grande quantidade de falha nesse período, também temos o Numero de tasks que falharam, então praticamente foram todas as tasks que falharam e não conseguiram dar continuidade mesmo com o retry.  

Praticamente todos esses gráficos seguem na mesma linha, o top 5 Dags/Tasks que estão com mais status de falhas, o Dag/Tasks com Status de Falha, é por fim o Monitor de Anomalia de Tasks com Falhas, para saber os picos de dia/hora que teve mais fluxo de falha.

![Imgur](https://i.imgur.com/agrPZUk.png)

---

## Como Monitorar nossas Filas do RabbitMQ ?

Para finalizar, precisamos entender também como analisar nossas filas, que também temos o Dashboard, basta acessar [**[Iris][RabbitMQ][Queues]**](https://ab-inbev.datadoghq.com/dashboard/axc-5g7-r2a?tpl_var_bifrost_environment=production&tpl_var_rabbitmq_queue=%2A&from_ts=1658422138754&to_ts=1658436538754&live=true).

Após acessar o Dashboard do RabbitMQ, podemos ver que é quase um padrão de todos os dashboards possuir as variáveis para filtrar. É nesse caso do RabbitMQ, o melhor a se fazer é filtrar pela fila específica, que você quer monitorar, caso você não filtrar pela fila que quer monitorar, vai acabar ficando muito poluído o gráfico. 

Podem ver que na imagem a baixo, eu filtrei pela fila ***iris_generic_client***, para ter uma visibilidade melhor.

![Imgur](https://i.imgur.com/yYHyf9Y.png)

---

Assim que entramos no Dashboard, conseguimos analisar o Status do RabbitMQ, é também o Consumo da Fila, o primeiro gráfico mostra a proporção de tempo que os consumidores de uma fila podem receber novas mensagens.

Já o segundo gráfico mostra a quantidade de mensagens na fila nos últimos 15 minutos. Então esse gráfico mostra sempre a quantidade de mensagens que chegaram na fila nesses últimos 15 minutos.

![Imgur](https://i.imgur.com/vyigKaX.png)

---

Já na segunda visão do dashboard conseguimos verificar os gráficos laranjas que significam a taxa de transferência da fila (média de mensagens por segundo).

O primeiro gráfico mostra o total de mensagens na fila, esse gráfico e interessante de observar pois conseguimos saber se a fila está perto de bater 1 milhão de mensagens na fila, pois sabemos que e o valor máximo que o RabbitMQ consegue entregar.

Também temos a quantidade de mensagens entregues, o rate de mensagens publicadas e por fim temos o número por segundo de mensagens entregues a clientes e confirmadas. Com todos esses gráfico conseguimos tirar insights muitos interessante de como está o funcionamento de nossa filas. 

![Imgur](https://i.imgur.com/4FSEYGp.png)

---

Por fim temos os Logs, que também são uma das fontes que podemos analisar, caso algo de estranho ou problemas venham acontecer com nossas filas.


![Imgur](https://i.imgur.com/vEo6DFi.png)

---


## Resumo da Observabilidade dos Dashboards

Em resumo dos Dashboards, conseguimos concluir que os dashboards dão diversos insights de como está funcionando nossos processos é ferramentas. Também conseguimos analisar é adiantar possíveis erros que possam vir acontecer, antes de acontecer.

Os dashboards apresentados nesse artigo, são os mais importantes para monitorar as ferramentas e processos da arquitetura 2.0, mas também não esqueça de verificar os outros dashboards, como por exemplo o dashboard [**[Iris][Databricks]**](https://ab-inbev.datadoghq.com/dashboard/v3s-9mv-5uh?tpl_var_databricks_cluster_name=%2A&tpl_var_databricks_resource_name=%2A&from_ts=1657831947196&to_ts=1658436747196&live=true), que está totalmente detalhado para os usuários obterem insights mais profundos de seus clusters é processos. 


### DataDog
Em nosso DataDog, também temos Notificações e Alertas, que realmente nos ajudam a obter informações o mais rápido possível, quando algum processo ou sistema quebrou, ou o aumento excessivo de CPU, Memória, Disco, Uptime, Etc...

Na opção de alertas do DataDog, que podemos entrar por esse [**[Link]**](https://ab-inbev.datadoghq.com/monitors/manage?q=Iris), podemos ver diversos alertas que fazem a notificação através do Microsoft Teams, como mostra a imagem a baixo.

![Imgur](https://i.imgur.com/HfhaLYf.png)

Além de fazer a notificação através do Microsoft Teams, podemos também criar notificações para outras plataformas como E-mail, Weebhook, Slack, etc... 

Com isso podemos personalizar as mensagens de notificações do jeito que achamos melhor. No nosso [**[Alertas]**](https://ab-inbev.datadoghq.com/monitors/manage?q=Iris), podemos ver que já temos diversas Notificações que são mandadas para nosso grupo de canal no Teams.

---

## Como criar uma Notificação personalizada. 

Agora para completar o artigo com sucesso, vamos aprender a criar nossa própria notificação. Antes de começar criar nossa própria notificação, precisamos pedir acesso no ambiente do DataDog, para o time que faz a liberação de acessos a plataforma do DataDog. [**[DevOps - Liberação de Permissão ao DataDog]**](https://dev.azure.com/AMBEV-SA/PLATAFORMA/_boards/board/t/PLATAFORMA%20Team/Customer%20Service).

Após a liberação de permissão no DataDog, podemos continuar, no exemplo a baixo, vou explicar como fazer a criação de uma Notificação para uma Fila do RabbitMQ, vamos lá.


Primeiramente vamos na página de [**[Monitoramento]**](https://ab-inbev.datadoghq.com/monitors/manage)  do DataDog, clicamos em **New Monitor**.

![Imgur](https://i.imgur.com/eWpD2i2.png)

Após clicar em **New Monitor**, você vai ser redirecionado para página para começar a escolher qual tipo de monitoramento você quer fazer.

No caso vou escolher métrica, para criar uma notificação em cima da Fila do RabbitMQ.

![Imgur](https://i.imgur.com/xCVAIoA.png)

Agora vamos setar nossas métricas, na primeira parte você vai escolher qual o método de detecção, no meu caso eu escolhi o **Threshold Alert**.

Já na ***2*** parte você precisa definir a métrica, no caso eu escolhi a métrica do ***rabbitmq.queue_messages***, que vai trazer a quantidade total de mensagens na fila, eu estou trazendo da fila do ***RabbitMQ***, que tem o nome de ***iris_credit_serasa_information***, que também faz uma agregação por ***max by***, que vai pegar todo o histórico dessa fila, que teve o pico de mensagens na fila.

![Imgur](https://i.imgur.com/QvAxPY6.png)

---

Agora na parte ***3***, você seta a condição do alerta, como está no print a baixo, pode ver que eu coloquei que a notificação será acionada assim que bater 500 mil msgs ou for maior que esse valor de msgs. A notificação será disparada se esse limite de msg não abaixar da média de 500 mil msgs nos últimos 10 minutos. 

Você pode setar tanto um **alerta** ou um **warning**, então você pode ter duas verificações, uma menor e outra maior.

Agora na parte ***4***, você vai colocar o nome da Notificação, importante você descrever um nome correto, que seja fácil de identificar na hora de você lê o alerta, por exemplo: ***[Iris][RabbitMQ] Queues - IrisCreditSerasaInformation***.

Você também precisa colocar a descrição do alerta, no caso da imagem a baixo, eu descrevi oque está acontecendo no alerta. Também passei alguns caminhos de verificação, para saber por onde começar verificar, no caso como é um alerta informativo, eu descrevi o mais abrangente possível. Mas você pode descrever do jeito que achar melhor.

No caso desse alerta, estamos recebendo a notificação via Microsoft Teams, podemos ver que no final do alerta temos o ***@teams-ChatOpsNotifications***, esse e o nosso canal que recebe nossas notificações do DataDog. Para saber como integrar o seu canal do Microsoft Teams com o DataDog, acesse esse [**[Link - Integração Teams]**](https://docs.datadoghq.com/integrations/microsoft_teams/).

![Imgur](https://i.imgur.com/6Gv3Fcu.png)

---

Para finalizar temos a etapa ***5***, que é a ultima. Nessa etapa podemos definir a prioridade da notificação, tags, é também podemos definir o permissionamento da notificação, no caso quem poderá alterar e editar essa notificação. 

E por fim temos a opção de **Test Notification**, que você vai enviar uma notificação teste para seu canal do Microsoft Teams, para saber se realmente esta conforme o esperado, antes de você criar.

![Imgur](https://i.imgur.com/CYuSYZ1.png)

---

Após você criar sua notificação, ela vai aparecer na lista de notificações, igual a imagem a baixo, mostrando o status da notificação. Passando o mouse em cima da notificação, você vai poder **editar**, **clonar**, **mutar**, e **deletar** o alerta criado.

![Imgur](https://i.imgur.com/sRrTcMr.png)


---

## Resumo das Notificações

A criação de notificações é muito simples e muito poderosa para nos ajudar no dia a dia. Vale ressaltar a importância de obter informações da nossas aplicações, garantindo assim a proatividade  das manutenções necessárias.

Não se limite notificações prontas dos Datadog, faça do jeito que mais funcione melhor para o seu processo, obtendo insights e sendo proativo. Qualquer erro ou dispersão de informações é importante para garantir o bom funcionamento da ingestão.


---

# Vídeo Tutorial

<iframe src="https://anheuserbuschinbev.sharepoint.com/sites/irisplatform/_layouts/15/embed.aspx?UniqueId=bd67da66-e61d-473b-9062-d46e1a64ac3f&embed=%7B%22ust%22%3Atrue%2C%22hv%22%3A%22CopyEmbedCode%22%7D&referrer=StreamWebApp&referrerScenario=EmbedDialog.Create" width="640" height="360" frameborder="0" scrolling="no" allowfullscreen title="Monitoramento, Notificações e Canais.mp4"></iframe>


Caso tenha ficado alguma dúvida ou tenha alguma sugestão de melhoria, favor criar um card em nosso board!

- [**Atendimento Board Iris**](https://dev.azure.com/AMBEV-SA/IRIS-Analytics%20Platform/_boards/board/t/AnalyticsTeam/Customer%20Service)