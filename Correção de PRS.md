# Fluxo de PR

> Importante: Todo novo dataset deverá passar primeiramente pela avaliação de um dos aprovadores da própria Torre e, em seguida, deverá ser aberto um card no board do time de PlataformOps solicitando a aprovação de PR (card Pull Request). Para atualizações e bugs, algoritmo será necessária apenas a aprovação dos aprovadores da própria Torre.
{.is-warning}

O novo fluxo consiste em dois processos:

### Novos Dataset´s: 
![Imgur](https://i.imgur.com/KMCrxqU.png)

1-	Os desnevolvedores criam, modificam os PR´s. Após a finalização de uma PR, eles notificam o líder ou avaliador técnico de sua respectiva equipe para aprovação.

2-	Após a aprovação, é imprescindível incluir a TAG "APROVADO" para que a equipe de Operações identifique as PRs prontas para correção. Além disso, o aprovador deve abrir uma solicitação no Board de Atendimento, indicando qual PR deve ser analisada e aprovada para PRODUÇÃO.


3-	É importante notar que existem dois fluxos para que o time de Ops verifique as PR:
a.	Aprovação de PR urgentes: Nesse caso, há uma fila de exceção que deve ser notificada por meio do Teams no grupo "PR urgentes". Isso se aplica às Pull Requests que tratam de correções de bugs em produção, questões críticas de segurança e PRs relacionadas a metas. Essas solicitações têm prioridade máxima no atendimento e devem ser resolvidas o mais rápido possível.

b.	Aprovação de PR “normais”. Para as demais PRs, o fluxo segue o mesmo processo das solicitações do Board. A ordem de atendimento segue o método FIFO (First in, First Out) e as PRs são tratadas em concorrência com as outras solicitações.


4-	Após a identificação por meio da TAG "Aprovado", o processo segue conforme o tipo de PR, sendo então encaminhado à equipe de OPS para verificação dos parâmetros correspondentes. Caso seja necessário algum ajuste na PR, o aprovador identificará tanto na PR em si quanto na solicitação aberta as pendências a serem corrigidas, e moverá o campo "Pendências". Nesse momento, cabe ao desenvolvedor responsável pela PR realizar os ajustes necessários e notificar sobre a correção. Após concluir os ajustes, é importante comunicar o término do processo. 

### Atualizações e Bugs:

1-	Os desnevolvedores criam, modificam os PR´s. Após a finalização de uma PR, eles notificam o líder ou avaliador técnico de sua respectiva equipe para aprovação.

2-	O Líder ou avaliador técnico terá a permissão para aprovar e mergear o PR se o mesmo apresentar “ok” para os aspectos apresentados abaixo: 
Obs: Este fluxo deve ser seguido para os processos de Ingestion e Transformation. Para as PR de Intelligence, a interação ocorre exclusivamente com os aprovadores designados em cada torre e com time do COE.


# Pontos Avaliados na correção

# Correção de PR - Ingestion 1.0

1.	Seguir os padões de nomenclatura na seguinte documentação 
https://iris.ambev.com.br/pt-br/iris-ingestion/standardization-of-nomenclature

2. Seguir padronização de partição 
```	
"year=2020"
"month=02"
"day=01"
write.format("parquet").mode("overwrite").partitionBy('year').save(consumezone/Brazil/Contexto/Dominio/Produto)
  ```

> | Do     | Don't          | 
> | ------------- |:-------------:| 
> | Spark       | Pandas | 
> | FileWriter() | Spark.write.format()  |   
> | Código em produção sem comentários | Comentários / Display / Print
> | spark.write(path=’consumezone/Brazil/commandcenter/…’) | spark.write(path='r2d2')   | 
> | dataframe.write.mode(“overwrite”). save(“/mnt/consumezone/Brazil/CommandCenter”) | dataframe.write.mode(“overwrite”). save(“/mnt/r2d2/Brazil/CommandCenter”)  |  
> | Código otimizado | Count / Collect |  
> | ADF batendo direto em API | API_PZ_relatorio_PPL  |  


## DATAFACTORY: 

**PPL**

As PPL devem está dentro da pasta datafactory/pipeline + a nomenclatura clara e _PPL.json 

 ![Imgur](https://i.imgur.com/AdBctj0.png)

No caso de de “timeout”: “7.00:00:00” o retry deve estar com o valor 0. 

 ![Imgur](https://i.imgur.com/dJgjcWV.png)

## TRIGGER:

Nomenclatura das Triggers 

A_[Domínio][Nome do Pipeline ou Pipelines] 

[Domínio][Nome do Pipeline ou Pipelines] 

Trigger com A_ será ligada somente no ambiente de prod 

Possíveis status de Triggers: 

A_ runtimeState = Stopped: Trigger ligada apenas em produção;

A_ runtimeState = Started: Trigger ligada em prod e dev (Se atentar e pedir para trocar para Stopped, para que a trigger não fique ligada nos dois ambientes, gastando recursos desnecessário;  

Trigger sem A_ com runtimeState = started: Trigger ligada apenas em dev;

Trigger sem A_ com runtimeState = Stopped: Trigger não será ligada nem em dev nem em prod. 

## DATASET

Nomenclatura: 

[Local]_[Sistema]_[Dataset]_DTS 
 

Obs: Para o time de supply existe uma exceção. 

Os sinks para o Synapse que devem conter "SNP" (quando o destino é para o SQL Pool) ou "SNPSTORAGE" (quando o destino é o storage do Serveless), nesse caso ficaria: 

-> Sink_SNP_[Sistema]_[Dataset]_DTS 
Exemplo: Sink_SNP_LMS_DimGrupo_DTS 
ou 
-> Sink_SNPSTORAGE_[Sistema]_[Dataset]_DTS 
Exemplo: Sink_SNPSTORAGE_LMS_DimGrupo_DTS" 

## Linked Services

Verificar se existe chaves expostas e pedir para usar KeyVault (Se ficar na dúvida com o tipo do LS se precisa usar KV abrir o ADF e verificar o tipo do LS se é necessário o uso de KV) 

![Imgur](https://i.imgur.com/kh3fKvR.png) 

Verificar se estão alterando os LS Padrões como: 

Databricks 

Blob 

DataLake 

Verificar se o Connect via integration runtime esta como AutoResolveIntegrationRuntime caso estejam alterando para outro tipo como: IntegrationRunTime, pedir para ajustar. 

## Documentação .yaml

Todo notebook que escreve na CZ deve ter documentação, mesmo que seja atualização desse notebook é necessário deixar o link da documentação na descrição do PR. 

A documentação deve esta dentro da pasta documentation. 

A nomenclatura deve refletir o nome do dataset 

O campo “name” deve ser igual ao ultimo path do datalakepath 

![Imgur](https://i.imgur.com/qHtEXV0.png)

Os campos “description”, “teamowner”, “dataowner” e “dataexpert” deve conter aspas duplas caso tenha algum caractere especial. 

Caracteres especiais: [@!#$%^&*()<>?\\|}{~:] 

Datalakepath deve conter o País/domínio/subdomínio de acordo com a planilha de domínio e subdomínios válidos. Planilha de nomenclaturas validas. 

Exemplo de preenchimento da documentação:

Checar linha de partição: não pode estar vazio ou ter n/a.  

 ![Imgur](https://i.imgur.com/H2vXFIF.png)

Importante lembrar que no campo “description” é necessário aspas duplas em caso de caracters especiais. 

Checar se todas as colunas têm descrição. 

É necessário também que a identação esteja correta e que a extensão do arquivo seja .yaml 

 

Para conferir melhor referente os principais erros na documentação .yaml consultar a documentação na Wiki:

https://iris.ambev.com.br/governance/purview-errors 

Importante também verificar no deploy a etapa do Validate onde é informado nos Warning os erros que existe na documentação. 
![Imgur](https://imgur.com/FhZcDg3.png)

## Databricks

### Notebook

Nomenclatura de notebooks 

Origem_Destino_Sistema_Dataset. 

Os datasets de qualidade de dados devem ser nomeados seguindo o seguinte padrão 

Quality_Sistema_Dataset 

(Tem alguns notebooks de monitoramentos e genéricos que o pessoal usa, deve ter alguma nomenclatura?) 

Não podemos ter notebooks que ler da PZ  escreve direto para CZ. 

Os notebooks que escrevem na CZ deve conter a documentação .yaml e o path de escrita deve ser o mesmo do datalakepath da documentação. 

Evitar o uso da biblioteca Pandas, pois a mesma pode sobre sobrecarregar o driver do Cluster. 

Utilizar a PyIris para leitura e escrita de datasets na CZ, DW e Presto. 

 

OBS: Para o time de Command Center, quando tiver a tag “Atualização” não cobrar a parte de nomenclatura. 

Para o time de LAS também não cobrar a parte de nomenclatura. 

**Tags**

Assim que o responsável por avalia os PRs de cada torre termina a avaliação, é necessário que coloque a tag “Status: Aprovado” só assim o time  Íris irá avaliação esse PR. 

![Imgur](https://i.imgur.com/AVutaw5.png)
 
Quando tiver as tags de “Bug” ou “Atualização” não cobrar ajuste de nomenclatura. 

 ![Imgur](https://i.imgur.com/GFLsRLd.png) 

Quando avaliar o PR e precisar de ajuste da parte dos times, retirar a tag “Status: Aprovado” e colocar a tag “Status: Alteração Pendente” 

 ![Imgur](https://i.imgur.com/eycK4r6.png)

Quando tiver a tag “Assunto: Novo Dataset” verificar se nomenclaturas esta correta e se o mesmo path que esta na documentação YAML é o mesmo path que esta sendo escrito na CZ. 
![Imgur](https://i.imgur.com/OTV3tez.png)



# Correção de PR - Ingestion 2.0

A imagem abaixo apresenta o processo de correção de Pull Requests. Um vídeo gravado também está disponível, que mostra os passos a passo e exemplos práticos de como aplicar esse processo no dia a dia de ingestão de dados.

![Imgur](https://imgur.com/6ar8nZM.jpg)


## Pontos Obrigatórios

Como pontos obrigatórios temos os seguintes tópicos, que se referem a imagem e o vídeo.

1. Scheduler: O scheduler é o ponto em que precisamos analisar se o cron informado pelo usuário é coerente com o tipo de conector utilizado. Para fazer essa consulta, você pode utilizar o site <a href="https://crontab.guru/examples.html" target="_blank">**Cron Tab - Guru**</a>. Para cada tipo de conector, temos a responsabilidade de utilizar o cron de forma coerente com nossas necessidades, sem desperdiçar recursos desnecessários.

2. Connection: É importante verificar se a conexão foi preenchida pelo usuário e se ela foi criada no Airflow. Existem já um tutoriais que ensinam como criar conexões no Airflow. No momento do PR, é recomendável analisar se o nome escolhido para a conexão faz sentido e o campo foi preenchidos.

3. Metadata: Sobre o metadata, é importante verificar se ele foi preenchido corretamente e se possui descrições coerentes das colunas informadas pelo usuário. Caso contrário, é necessário retornar ao usuário para que ele faça as correções necessárias.

4. Webhook: O webhook é o nosso meio de comunicação em caso de falha do pipeline. Por isso, é obrigatório que o usuário preencha e crie seu webhook conforme a documentação detalhada na parte de Ingestão. Caso o pipeline falhe, o webhook enviará uma mensagem ao canal do Teams informando o erro ao usuário.

5. Geração de aplicação: A geração da pasta aplication é executada no momento do commit do branch do usuário. É obrigatório que todas as modificações feitas pelo usuário no caminho '/domain' sejam geradas. Sem essas gerações, o código não poderá ser lido pelo Airflow, pois não foi transpilado para a nomenclatura de leitura do Airflow. Portanto, é obrigatório que você verifique se o arquivo foi gerado na pasta '/application'.

6. Geração de metadados: A geração de metadados segue o mesmo processo do tópico "5". O que você precisa fazer é verificar se os arquivos corretos foram gerados na pasta '/application', conforme mencionado no tópico acima.



## Pontos de atenção

Como pontos de atenção para o funcionamento da DAG em produção, existem dois tópicos essenciais que devem ser observados.

1. **NUNCA** modifique a pasta application: Atualmente, não é possível modificar os arquivos da pasta '/application' **manualmente**. Se você tentar fazer uma alteração na pasta '/application' sem modificar o pipeline na pasta '/domain', as alterações serão perdidas na próxima geração da fábrica. Isso é incorreto e será abordado no tópico de estrutura de pastas.

2. Verificar se a DAG foi gerada: Este passo é essencial para o funcionamento da DAG. A DAG é sempre gerada quando o usuário faz um commit, pela fábrica, até o momento atual. Mas como verificamos isso? No vídeo abaixo, mostramos que sempre que houver modificações nas pastas '/domain' e '/application' e essas modificações forem coincidentes, significa que a DAG foi gerada corretamente e não haverá problemas. O problema ocorre quando o usuário altera apenas a pasta '/application' ou apenas a pasta '/domain'. Portanto, lembre-se de que é sempre necessário que ambas as pastas sejam modificadas e que as modificações sejam coincidentes.


## Estrutura de Pastas

A figura acima mostra a estrutura de pastas do repositório <a href="https://github.com/cervejaria-ambev/correnteza" target="_blank">**Correnteza**</a>. É importante entender que, sempre que houver um novo dataset (pipeline), é necessário criar as pastas metadata e pipeline. Essas pastas são onde serão criados o metadado (doc) e a DAG (pipeline), temos também comandos especificos que já fazem essa criação automaticamente, na documentação de ingestão.

Outro fator importante para o sucesso da correção do Pull Request é entender que as alterações na pasta '/application' devem ser feitas automaticamente pela fábrica. Isso ocorre sempre que um usuário faz um commit na sua branch ou quando o comando **make generate-dags** é executado. Se houver alterações apenas nos arquivos da pasta '/application', elas serão removidas quando outro usuário rodar a fábrica.

Por isso, o usuário deve sempre fazer suas alterações na pasta '/domain', onde está o pipeline em questão. Alterações em outras pastas, ou em pipelines de outros usuários, não serão aprovadas. Lembre-se de sempre aprovar e mesclar alterações que sejam apenas do usuário e que estejam de acordo com o que ele está fazendo.

## Bônus - Boas Práticas

Na seção de bônus, temos alguns aspectos que são boas práticas, mas que alguns usuários não seguem. Seguir essas boas práticas pode poupar muito tempo e evitar erros.


1. Realizar Teste Localmente: O teste local é uma ferramenta facilitadora que pode evitar erros comuns, como erros de sintaxe, caminho errado ou conexão incorreta. Sempre recomendo que os usuários executem o teste local antes de enviar um Pull Request. Isso economizará tempo e evitará erros bobos. <a href="https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/ingest#testando-localmente" target="_blank">**Documentação Teste Localmente**</a>.

2. Validar dados Dabricks [Lake]: É essencial e uma boa prática que os usuários validem os dados no lake após a ingestão em produção. Isso ocorre porque, às vezes, os usuários podem ingerir dados e, durante a análise, perceber que faltam colunas, há dados incorretos ou é necessário realizar alguma transformação. A validação dos dados pode ajudar a evitar esses problemas e evitar estresse e perda de tempo. <a href="https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/databricks" target="_blank">**Documentação Valide os dados no Lake utilizando Databricks**</a>.

## Vídeo Tutorial

Vídeo prático sobre como fazer correções de Pull Requests. Assista ao vídeo para saber mais.

<iframe src="https://anheuserbuschinbev.sharepoint.com/sites/irisplatform/_layouts/15/embed.aspx?UniqueId=8e9394af-4f4d-4b34-a054-ac396bc54d0c&embed=%7B%22ust%22%3Atrue%2C%22hv%22%3A%22CopyEmbedCode%22%7D&referrer=StreamWebApp&referrerScenario=EmbedDialog.Create" width="640" height="360" frameborder="0" scrolling="no" allowfullscreen title="Correção de PRs Correnteza.mp4"></iframe>


# Correção de PR - Transformation 

## Descrição do Pull Request

1) O pull request apresenta descrição clara e objetiva?

2) A opção de alteração é coerente? Ou seja, caso marcado o PR como um bug, a opção escolhida está correta?

3) A label inserida condiz com sua alteração?

4) Foram verificados todos os pontos do checklist?

## Verificações na esteira de CI/CD
1) Todas as verificações de qualidade, estilo e demais testes executaram com sucesso no [pipeline de CI/CD](https://dev.azure.com/AMBEV-SA/IRIS-AnalyticsPlatform/_build?definitionId=11454)?


## Metadados
As verificações abaixo estão relacionadas com o arquivo .yaml de documentação.

1)	O campo de descrição foi preenchido corretamente?
2)	O Campo dataexpert foi preenchido com o nome do responsável/criador do dataset?
3)	As descrições de cada um dos campos estão claras?
4)	Os tipos dos campos condizem com as descrições? Ex: Um campo chamado de data_de_entrega está como date ou datetime?
5) Seu dataset possui dados sensíveis ou pessoais? Caso sim, anexe as evidências do processo de One Trust
6) Os campos estão em formato snake_case e sem caracteres especiais?

## Pipeline
As verificações abaixo estão relacionadas com o arquivo .yaml de seu pipeline.

1)	O campo <b>dag_class</b> está preenchido com o valor <b>analytical</b>?
2)	O campo <b>dag_type</b> está preenchido com o valor <b>transformation</b>?
3)	O campo <b>schedule</b> possui um cron válido? Por favor, verifique através desse [site](https://crontab.guru/)
4)	O campo <b>country</b> possui uma das opções válidas abaixo?
- "Argentina"
- "Bolivia"
- "Brazil"
- "Chile"
- "Colombia"
- "Ecuador"
- "Guyana"
- "Paraguay"
- "Peru"
- "Suriname"
- "Uruguay"
- "Venezuela"
- "Saz"
5)	Os campos <b>domain</b>  e <b>sub-domain</b> possuem uma das opções válidas abaixo?

<table style="border-collapse: collapse; width: 100%;">
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px; font-weight: bold;">
        <td>Domain</td>
        <td>Sub-domain</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>BREWDAT</td>
        <td>SALES</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>COE</td>
        <td>TECHANALYTICS</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>CONSUMER</td>
        <td>DIRECTTOCONSUMER</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>CONSUMER</td>
        <td>INNO</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>CONSUMER</td>
        <td>MARTECH</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>CONSUMER</td>
        <td>TRADE</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>DATAGOVERNANCE</td>
        <td>DATAQUALITY</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>DATAGOVERNANCE</td>
        <td>IRIS</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>ENTERPRISEINTELLIGENCE</td>
        <td>CADASTRO</td>
    </tr>
     <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>ENTERPRISEINTELLIGENCE</td>
        <td>INDICADORESAGILIDADE</td>
    </tr>
     <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>ENTERPRISEINTELLIGENCE</td>
        <td>INDICADORESPRODUTO</td>
    </tr>
       <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>ENTERPRISEINTELLIGENCE</td>
        <td>NORMALIZACAO</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>ATR</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>TESOURARIA</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>HERA</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>CAPEX</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>ANAPLAN</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>CASHFLOW</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>BALANCE</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>PNL</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>CAPEXOBZ</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>VILC</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>EXTERNALREPORTING</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>INTERNALREPORTING</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>OVERHEAD</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FINANCE</td>
        <td>FX</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FPNA</td>
        <td>BUSINESSCYCLE</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FPNA</td>
        <td>CAPEX</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FPNA</td>
        <td>CASHFLOW</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FPNA</td>
        <td>MONTHLYCYCLE</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FPNA</td>
        <td>OVERHEAD</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FPNA</td>
        <td>BREWDAT</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FPNA</td>
        <td>ANAPLAN</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FPNA</td>
        <td>MONITORING</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FPNA</td>
        <td>HERA</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>FPNA</td>
        <td>CAPEX</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LEGAL</td>
        <td>CONTENCIOSO</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LEGAL</td>
        <td>CORPORATE</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/BEES</td>
        <td>AVAILABEE</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/BEES</td>
        <td>DISPONIBILIDADEESTOQUE</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/BEES</td>
        <td>LOAT</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/BEES</td>
        <td>ORDERS</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/DISTRIBUTION</td>
        <td>AS</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/DISTRIBUTION</td>
        <td>CUSTOS</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/DISTRIBUTION</td>
        <td>DIMENSION</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/DISTRIBUTION</td>
        <td>ENTREGA</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/DISTRIBUTION</td>
        <td>PEDIDOS</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/DISTRIBUTION</td>
        <td>PRODUTIVIDADE</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/DISTRIBUTION</td>
        <td>ROTEIRIZADOR</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/DISTRIBUTION</td>
        <td>SIZING</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/DISTRIBUTION</td>
        <td>OBAMA</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/PLANNING</td>
        <td>AS</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/PLANNING</td>
        <td>BUDGET</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/PLANNING</td>
        <td>CDP</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/PLANNING</td>
        <td>CONA</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/PLANNING</td>
        <td>DEMANDA</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/PLANNING</td>
        <td>DSNP</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/PLANNING</td>
        <td>INFULL</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/PLANNING</td>
        <td>INSUMOS</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/PLANNING</td>
        <td>MARKETPLACE</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/PLANNING</td>
        <td>PRODUCAO</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>LOGISTICS/PLANNING</td>
        <td>VLC</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>LOGISTICS/PLANNING</td>
    <td>WSNP</td>
</tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>LOGISTICS/TRANSPORT</td>
      <td>DIMENSION</td>
  </tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>LOGISTICS/TRANSPORT</td>
      <td>EXECUTION</td>
  </tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>LOGISTICS/TRANSPORT</td>
      <td>TACTICAL</td>
  </tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>LOGISTICS/TRANSPORT</td>
      <td>TIMELINE</td>
  </tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>LOGISTICS/WAREHOUSE</td>
      <td>CONTROLE</td>
  </tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>LOGISTICS/WAREHOUSE</td>
      <td>MARKETPLACE</td>
  </tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>LOGISTICS/WAREHOUSE</td>
      <td>T1</td>
  </tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>LOGISTICS/WAREHOUSE</td>
      <td>T2</td>
  </tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>MATCH</td>
      <td>CONTROLTOWER</td>
  </tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>MATCH</td>
      <td>SELLIN</td>
  </tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>MATCH</td>
      <td>SELLOUT</td>
  </tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>MATCH</td>
      <td>DTC</td>
  </tr>
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
      <td>MATCH</td>
      <td>NPS</td>
  </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>PEOPLE</td>
        <td>CULTURA</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>PEOPLE</td>
        <td>DATAENGINEERINGMONITOR</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>PEOPLE</td>
        <td>DIVERSIDADEINCLUSAO</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>PEOPLE</td>
        <td>FINANCEIRO</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>PEOPLE</td>
        <td>GESTAOFUNCIONARIOS</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>PEOPLE</td>
        <td>GESTAOFUNCIONARIOS</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>PEOPLE</td>
        <td>PLANODECARREIRA</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>PEOPLE</td>
        <td>PLATFORMINSIGHTS</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>PEOPLE</td>
        <td>RECRUTAMENTOESELECAO</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>PEOPLE</td>
        <td>SEGURANCA</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
  <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PEOPLE</td>
    <td>TRACKINGDETALENTOS</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>AGRO</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>ALLOCATION</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>BEES</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>CONTRATOS</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>COPRODUTOS</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>ESTOQUE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>FROTA</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>INSUMOS</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>MATERIAIS</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>PACKAGING</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>PAGAMENTOS</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>PURCHASEDATA</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>PROCUREMENT</td>
    <td>SUSTENTABILIDADE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>AURORA</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>BEES</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>CORA</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>CORE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>CUSTOMEREXPERIENCE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>DIGITALINSIGHTS</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>FINANCE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>FULLDIGITAL</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>INNOTRADE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>INTELIGENCIADEMERCADO</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>LAB</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>OFFTRADE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>ONTRADE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>REVENUEMANAGEMENT</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>MARKETPLACE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>NAB</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SALES</td>
    <td>SISTEMAS</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SUPPLY</td>
    <td>ESTRUTURA</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SUPPLY</td>
    <td>GENTE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SUPPLY</td>
    <td>GLOBALKPIS</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SUPPLY</td>
    <td>MONITORAMENTO</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SUPPLY</td>
    <td>PRODUCAO</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SUPPLY</td>
    <td>SEGURANCA</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SUPPLY</td>
    <td>SOCIALMEDIA</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SUPPLY</td>
    <td>PRODUCTKPIS</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>SUPPLY</td>
    <td>FINANCEIRO</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>TAX</td>
    <td>FINTAX</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>TAX</td>
    <td>COMPLIANCE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>TAX</td>
    <td>IMPOSTOS</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>TAX</td>
    <td>BACKOFFICE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>TECHOPS</td>
    <td>ATENDIMENTO</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>TECHOPS</td>
    <td>AUDITORIA</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>TECHOPS</td>
    <td>DIMENSAO</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>TECHOPS</td>
    <td>GOVERNANCE</td>
</tr>
<tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
    <td>TECHOPS</td>
    <td>MONITORAMENTO</td>
</tr>
 
</table>



6) O campo <b>context</b> faz sentido para o dataset criado? 
7)	O valor de <b>execution_timeout</b> está entre <b>10</b> e <b>600</b> minutos?
8)	Os campos <b>owner</b>, <b>owner_id</b> e <b>owner_email</b> fazem referência ao responsável/criador do dataset?
9)	O nome do <b>dataset</b> faz sentido e é o mesmo da classe da transformação?
10)	O campo <b>mode</b> do sync é <b>append_merge</b> ,<b>append</b>, <b>append_drop</b>, <b>overwrite</b>?
11)	O campo <b>partition_by</b> é opcional. Caso necessite particionar seu dataset resultante da transformação, esse campo foi indicado como uma lista de valores, conforme demonstrado abaixo?
```
partition_by:
	- "data_emissao"
```
13)	No campo de dependência <b>depends_on</b>, as dependências externas são válidas?
- external.dag = Conferir o nome da Dag no Airflow de prod.
- external.dataset = Conferir o nome do dataset no Airflow de prod.
14)	No campo de dependência <b>depends_on</b>, as dependências externas são válidas?
- external.dag = Conferir o nome da Dag no Airflow de prod.
- external.dataset = Conferir o nome do dataset no Airflow de prod.
15) A chave serving.adhoc.active apresenta valor booleano.
o)  O campo de “metadata” está com o nome correto do arquivo de documentação?

## Databricks notebook source
Aqui vamos verificar se o arquivo .py gerado no Databricks (que deve ser colcado na pasta `/transformation`) apresentas os pré requisitos necessários.

1. Possui na primeira linha o comentário necessário para o Databricks entender que é um notebook?
```
# Databricks notebook source
```

2. Seu código apresenta as importações de bibliotecas essenciais?
```
import json
from pyiris.transformation.task.task_entrypoint import TaskEntryPoint
from pyiris.transformation.transformation import Transformation
from pyspark.sql import DataFrame
```

3. O nome da classe apresenta o nome do dataset no formato CamelCase? A classe está herdando a classe transformation?
```
# Databricks notebook source
import json
from pyiris.transformation.task.task_entrypoint import TaskEntryPoint
from pyiris.transformation.transformation import Transformation
from pyspark.sql import DataFrame

# Main class for your transformation
class MyDatasetName(Transformation):
		def __init__(self):
        super().__init__(
            ...
        )
...
```

4. No construtor da classe, existe o parâmetro `dependencies` sendo passado na inicialização da classe mãe? Conforme exemplo a seguir:
```
...
class MyDatasetName(Transformation):
    def __init__(self):
        super().__init__(
            dependencies=[
                {
                    "table_id": "my_dependency_table_id",
                    "country": "Brazil",
                    "path": "path/to/dependency",
                    "zone": "trustedzone",
                    "format": "delta",
                }
            ]
        )
...
```
        

5. Os métodos da classe do usuário estão com doc string e a tipagem de retorno?
![Imgur](https://i.imgur.com/cR3frBt.png =700x){.align-center}

6. Existe código comentado na classe/métodos do usuário?
7. A classe de transformação possui o método definitions?
8. O método definitions está retornando o dataframe que deverá ser persistido no lake?
9. Os métodos possuem responsabilidades e objetivos claros?
10. O notebook contém o "rodapé" com as chamadas do `task_entry_point` definidos como ultimas linhas?
```
...
context = json.loads(dbutils.widgets.get("context"))
task_entry_point = TaskEntryPoint(
    context=context, transformation_object=MyDatasetName()
)
task = task_entry_point.handle()
task.run()
```

11. Na chamada do task_entry_point é referenciada a classe criada no Notebook?
![Imgur](https://i.imgur.com/D5vdK5r.png =700x){.align-center}


## Vídeo Tutorial
Após revisar toda a documentação, você ainda tem alguma dúvida sobre os procedimentos ou cenários abordados? Para auxiliar na compreensão dos passos necessários e do processo em geral, foram preparados alguns vídeos explicativos. Estes vídeos fornecem uma visão ampla da correção de um PR de um novo dataset e  de um fix. Se as dúvidas persistirem, recomendamos assistir aos tutoriais em vídeo disponibilizados abaixo. Estamos à disposição para esclarecer qualquer questionamento adicional.

#### Entendendo a correção de um Novo Dataset

<iframe src="https://anheuserbuschinbev.sharepoint.com/sites/irisplatform/_layouts/15/embed.aspx?UniqueId=ba58601b-251e-4ece-8090-a374a8100ae6&embed=%7B%22ust%22%3Atrue%2C%22hv%22%3A%22CopyEmbedCode%22%7D&referrer=StreamWebApp&referrerScenario=EmbedDialog.Create" width="640" height="360" frameborder="0" scrolling="no" allowfullscreen title="Entendendo a correção de um novo dataset.mp4"></iframe>

#### Entendendo a correção de um FiX

<iframe src="https://anheuserbuschinbev.sharepoint.com/sites/irisplatform/_layouts/15/embed.aspx?UniqueId=bd91d24c-e2ec-4bf6-a59e-b31ee4f68cbb&embed=%7B%22ust%22%3Atrue%2C%22hv%22%3A%22CopyEmbedCode%22%7D&referrer=StreamWebApp&referrerScenario=EmbedDialog.Create" width="640" height="360" frameborder="0" scrolling="no" allowfullscreen title="Entendendo a correção de um fix.mp4"></iframe>