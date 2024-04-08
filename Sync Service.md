# O que é o Sync Service?

O serviço de sincronização, conhecido como **sync service**, desempenha um papel crucial na harmonização de dados no lake. Através dele, temos a flexibilidade de escolher estratégias que **orientarão** como os dados serão **gravados** no lake.

Dentro da plataforma Iris, há a possibilidade de empregar quatro tipos distintos de estratégias para sincronização de dados no lake: *overwrite*, *append_merge*, *append_drop* e *append_only*. Ao selecionar a estratégia adequada para a sua transformação, basta especificá-la no parâmetro **'mode'** do arquivo YAML.

> **Informação**: Essa documentação é válida tanto para os produtos de **Ingestão** quanto para o de **Transformação**! Portanto, uma compreensão sólida dos conceitos trará benefícios significativos para a utilização de ambos os produtos!
{.is-info}


# Estratégia overwrite

A estratégia  overwrite permite a evolução do schema por meio da habilitação da configuração do delta **OverwriteSchema** como **True**. Essa configuração possibilita alterar o schema dos dados de seu dataset sem ocasionar um erro de incompatibilidade entre os schemas. Além disso, ela facilita quando precisamos fazer um reprocessamento de dados com um schema diferente, uma vez que ele será sobrescrito.


## Como funciona a estratégia overwrite?

A estratégia overwrite permitirá a alteração dos tipos de dados nas mesmas colunas (por exemplo: **String para Integer**). Essa estratégia  possibilita **descartar** uma coluna, alterar o **tipo** de dados da coluna existente e/ou **renomear** nomes de colunas que diferem apenas por **maiúsculas** e **minúsculas**.

- **Cenário 1:** Mudanças de Tipo, Exemplo: **(IntegerType para StringType)**
- **Cenário 2:** Mudança de nome da coluna em maiúsculas e minúsculas, Exemplo: **(Foo para foo)**
- **Cenário 3:** **Remover** coluna
- **Cenário 4:** **Alterar** coluna

Em essência, a estratégia **overwrite** se destaca como a escolha ideal quando há a necessidade de modificar o tipo de dado em uma coluna, como por exemplo, de **String para Integer.** No entanto, é importante destacar que essa abordagem pode ser aplicada em diversos cenários, conforme mencionado anteriormente.

Vamos verficar um exemplo prático! Meu dataset inicial apresenta **4 colunas**: 

<table style="border-collapse: collapse; width: 100%;">
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px; font-weight: bold;">
        <td>codigo_filial</td>
        <td>quantidade_maxima</td>
        <td>codigo_produto</td>
        <td>year</td>
    </tr>
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>541</td>
        <td>1</td>
        <td>4293</td>
        <td>2023</td>
    </tr>
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>538</td>
        <td>77</td>
        <td>838</td>
        <td>2023</td>
    </tr>
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>843</td>
        <td>1</td>
        <td>9274</td>
        <td>2023</td>
    </tr>
</table>


**Cenário**: Preciso alterar meu dataset para adicionar mais **1 coluna**. 

Utilizando essa estratégia será permitido fazer esse tipo de alteração do schema de dados sem se preocupar em deletar os dados direto no path, o resultado seria o dataset abaixo, **observe a marcação em azul**: 

<table style="border-collapse: collapse; width: 100%;">
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px; font-weight: bold;">
        <td>codigo_filial</td>
        <td>quantidade_maxima</td>
        <td>codigo_produto</td>
        <td>year</td>
        <td style="background-color: #a6cdf2;">group</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>541</td>
        <td>1</td>
        <td>4293</td>
        <td>2023</td>
        <td style="background-color: #a6cdf2;">1</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>538</td>
        <td>77</td>
        <td>838</td>
        <td>2023</td>
        <td style="background-color: #a6cdf2;">1</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>843</td>
        <td>1</td>
        <td>9274</td>
        <td>2023</td>
        <td style="background-color: #a6cdf2;">1</td>
    </tr>
</table>

### Como utilizar a estratégia overwrite?

Para utilizar essa estratégia basta passá-la dentro do **yaml** o parâmetro mode do sync.

***Dê uma olhada no exemplo abaixo.:***

```
...
datasets:
	...
	sync:
		mode: "overwrite"
	...
...

```

# Estratégia append merge

A estratégia append merge permite a evolução do schema habilitando a configuração do delta **MergeSchema** como **True**. Com isso, será permitida uma evolução do schema durante a mesclagem do dados. Isso significa que, se uma coluna estiver presente na tabela de origem, mas não na tabela de destino, a nova coluna será adicionada à tabela de destino e seus valores serão inseridos ou atualizados usando os valores da fonte.


## Como funciona a estratégia append merge?

**MergeSchema** é utilizado nos seguintes cenários:

- **Cenário 1**: Adicionando **novas** colunas *(este é o cenário mais comum)*
- **Cenário 2**: Mudança do tipo de dado de **non-nullable** para **nullable**
- **Cenário 3**: Upcasts from **ByteType** -> **ShortType** -> **IntegerType**

O **mergeSchema**, você apenas faz **append**, então é o modo **mais** seguro de se utilizar e que não precisa fazer a exclusão do seu diretorio. Básicamente o melhor cenário de se utilizar o **append_merge**, é quando você precisa **incluir uma nova coluna**. Essa nova coluna quando é adicionada, passa os dados como **nulos** aonde **não possui aquele dado em específico**. 

Ao utilizar a estratégia de **append_merge**, quaisquer colunas que estejam presentes no DataFrame, mas **não** na tabela de destino, são automaticamente **adicionadas** ao final do esquema como parte de uma *transação de escrita*, como no exemplo abaixo.

<table style="border-collapse: collapse; width: 100%;">
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px; font-weight: bold;">
        <td>codigo_filial</td>
        <td>quantidade_maxima</td>
        <td>codigo_produto</td>
        <td>year</td>
    </tr>
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>541</td>
        <td>1</td>
        <td>4293</td>
        <td>2023</td>
    </tr>
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>538</td>
        <td>77</td>
        <td>838</td>
        <td>2023</td>
    </tr>
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>843</td>
        <td>1</td>
        <td>9274</td>
        <td>2023</td>
    </tr>
</table>


Depois da inclusão de uma **nova coluna**, o nosso conjunto de dados terá a seguinte configuração, **observe a marcação em azul**:

<table style="border-collapse: collapse; width: 100%;">
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px; font-weight: bold;">
        <td>codigo_filial</td>
        <td>quantidade_maxima</td>
        <td>codigo_produto</td>
        <td>year</td>
     		<td style="background-color: #a6cdf2;">country</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>541</td>
        <td>1</td>
        <td>4293</td>
        <td>2023</td>
      	<td style="background-color: #a6cdf2;">null</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>538</td>
        <td>77</td>
        <td>838</td>
        <td>2023</td>
      	<td style="background-color: #a6cdf2;">null</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>843</td>
        <td>1</td>
        <td>9274</td>
        <td>2023</td>
      	<td style="background-color: #a6cdf2;">null</td>
    </tr>
      <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>2</td>
        <td>1</td>
        <td>3435</td>
        <td>2023</td>
      	<td style="background-color: #a6cdf2;">Brazil</td>
    </tr>
</table>

>***Atenção:*** Como tabela a forma antiga não apresentavam a coluna country, os registros antigos serão setados por default como nulos pelo próprio spark.
{.is-warning}

### Como utilizar a estratégia append merge?

Para utilizar essa estratégia basta passá-la dentro do **yaml** o parâmetro mode do sync.

***Dê uma olhada no exemplo abaixo.:***

```
...
datasets:
	...
	sync:
		mode: "append_merge"
	...
...

```

# Estratégia append drop

A estratégia de **append_drop** analisará se existe alguma **diferença** entre o esquema de dados no lake e o da transformação. Se houver divergências, como um número distinto de colunas, o sistema eliminará completamente o conjunto de dados existente no lake e substituirá pelo novo conjunto de dados. Em outras palavras, todo o **histórico** será **excluído** para, posteriormente, realizar a **adição** dos novos dados por meio de append.

## Como funciona a estratégia append drop?

- **Cenário 1:** Recomendado para situações em que o **esquema** de dados sofre **mudanças frequentes** *(Particularmente útil em processos de transformação).*

**Cenário:** A rotina do meu pipeline todo o dia **insere** os dados com schema da tabela abaixo, porém preciso adicionar uma **nova coluna** ao meu dataset e **desconsiderar** todo o histórico dos dados para fazer um reprocessamento: 

<table style="border-collapse: collapse; width: 100%;">
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px; font-weight: bold;">
        <td>codigo_filial</td>
        <td>quantidade_maxima</td>
        <td>codigo_produto</td>
        <td>year</td>
    </tr>
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>541</td>
        <td>1</td>
        <td>4293</td>
        <td>2023</td>
    </tr>
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>538</td>
        <td>77</td>
        <td>838</td>
        <td>2023</td>
    </tr>
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>843</td>
        <td>1</td>
        <td>9274</td>
        <td>2023</td>
    </tr>
</table>

Após executar meu processo, meu **novo** dataset ficará da seguinte forma, **observe a marcação em azul**:

<table style="border-collapse: collapse; width: 100%;">
    <tr style="border-bottom: 1px solid #ddd; text-align: left; padding: 8px; font-weight: bold;">
        <td>codigo_filial</td>
        <td>quantidade_maxima</td>
        <td>codigo_produto</td>
        <td>year</td>
     		<td style="background-color: #a6cdf2;">country</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>23</td>
        <td>1</td>
        <td>423293</td>
        <td>2023</td>
      	<td style="background-color: #a6cdf2;">Brazil</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>54438</td>
        <td>77</td>
        <td>83328</td>
        <td>2023</td>
      	<td style="background-color: #a6cdf2;">Brazil</td>
    </tr>
    <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>83443</td>
        <td>1</td>
        <td>9374</td>
        <td>2023</td>
      	<td style="background-color: #a6cdf2;">null</td>
    </tr>
      <tr style="background-color: #f2f2f2; border-bottom: 1px solid #ddd; text-align: left; padding: 8px;">
        <td>2</td>
        <td>1</td>
        <td>3435</td>
        <td>2023</td>
      	<td style="background-color: #a6cdf2;">Brazil</td>
    </tr>
</table>

Dessa forma, a minha dag **deletou automaticamente** todo o meu histórico e inseriu novos dados com um novo **schema**.

### Como utilizar a estratégia append drop?

Para utilizar essa estratégia basta passá-la dentro do **yaml** o parâmetro mode do sync.

***Dê uma olhada no exemplo abaixo.:***

```
...
datasets:
	...
	sync:
		mode: "append_drop"
	...
...

```

>***Atenção:*** A diferença fundamental entre **append_merge** e **append_drop** reside no fato de que a primeira é aconselhada quando é **crucial** preservar o **histórico de dados**, mesmo diante da adição de novas colunas. Enquanto isso, a segunda é a opção ideal quando a necessidade é **deletar** os dados ao modificar o **esquema do conjunto de dados**.
{.is-warning}

# Estratégia append only

A estratégia **append_only** é automaticamente definida como a estratégia padrão para a ingestão de dados. Essa abordagem realiza exclusivamente operações de append, empilhando dados sobre dados. Portanto, quando o usuário não especifica uma estratégia no pipeline de ingestão, a estratégia **append_only** é automaticamente aplicada como padrão. Em resumo, ela é a escolha **predefinida** quando não for feito a escolha do modo de sincronização na **ingestão**.

### Como utilizar a estratégia append only?

Para utilizar essa estratégia, é necessário especificá-la no arquivo yaml, utilizando o parâmetro mode do sync. No entanto, no caso do append_only, você tem a opção de não fornecer nenhum sync; nesse cenário, ele automaticamente define o append_only como padrão. Alternativamente, você também pode especificá-lo explicitamente, se preferir.

***Dê uma olhada no exemplo abaixo.:***

```
...
datasets:
	...
	sync:
		mode: "append_only"
	...
...

```

# Configurações Customizável e Defaults

Agora que você compreendeu os modos de sincronização mencionados anteriormente, como *overwrite*, *append_merge*, *append_drop* e *append_only*, está apto a selecionar entre as configurações personalizáveis ou as configurações padrão!

Dessa forma, proporcionamos total flexibilidade no comportamento das configurações do Delta, permitindo que você as ajuste conforme necessário.

Deixamos duas opções de configurações, pois o delta lake oferece configurações que permitem aos usuários controlar o comportamento e o desempenho das tabelas. Essas configurações podem ser aplicadas no nível da tabela, da sessão ou do sistema, influenciando tanto a leitura quanto a escrita de dados. Nesse cenário, disponibilizamos para que os usuários possam escolher as configurações adequadas para seus conjuntos de dados, aplicando tais configurações a nível da sessão do spark. 

## Configurações Defaults

Inicialmente, optamos por manter as configurações padrão exclusivamente configuradas para o ambiente de **transformation**. Até o momento, o ambiente de **ingestion** não possui configurações padrão **predefinidas**. No entanto, caso deseje, você tem a opção de criar configurações *personalizadas*. A decisão fica totalmente a critério do *usuário*. Importante notar que, no momento, apenas as DAGs de transformações possuem configurações padrão, que já são utilizadas sem você necessariamente decidir, para ganho de performance.

## Configurações Customizável

As configurações customizáveis, como o próprio nome sugere, permitem que você mesmo forneça suas **configurações do Delta Spark,** personalizando assim o pipeline de acordo com suas preferências e otimizando as configurações. Essa responsabilidade recai totalmente sobre o **usuário**! Portanto, é crucial compreender completamente o que está sendo **configurado** e estudar a fundo o caso antes de fornecer configurações que possam **impactar** seus dados e o desempenho!

***Dê uma olhada no exemplo abaixo.:***

```
...
datasets:
	...
    sync:
      mode: "append_only"
      vacuum_config:
        retain_hours: 1
      delta_config:
        custom_config: 
          'spark.databricks.delta.properties.defaults.autoOptimize.autoCompact': 'True'
          'Sua.Delta.Spark.Config.Exemplo...': 'True'
          'Sua.Delta.Spark.Config.Exemplo...': 'False'
	...
...

```

Esse exemplo e semelhante do anterior, o que realmente muda é a **custom_config**.

Basicamente a inclusão do parâmetro **"mode"** é *obrigatória*! Enquanto isso, a opção **"vacuum_config"** é opcional. Caso você decida fornecer o parâmetro **"vacuum_config"**, estará ativando a funcionalidade de **"vacuum"** em seu conjunto de dados. Para mais informações, você pode ler mais sobre o assunto. [**Vacuum**](https://delta.io/blog/2023-01-03-delta-lake-vacuum-command/).

Agora, em relação ao parâmetro **"delta_config"** neste exemplo, estamos optando pelo modo customizável! Aqui, você pode ajustar as configurações do **Delta** de acordo com suas preferências.

Para utilizar a **"custom_config"**, é possível incluir diversas configurações dentro das chaves de **"custom_config"**, conforme exemplificado acima. No entanto, é crucial ter cautela para não utilizar excessivas configurações que, em vez de melhorar o desempenho, possam prejudicar ou afetar seus dados adversamente.

> ***Atenção:*** Por padrão, o vacuum permite ser executado com um período de retenção maior ou igual a 168 horas (7 dias). Para utilizar um período menor, deve-se habilitar a config: **spark.databricks.delta.retentionDurationCheck.enabled** como **False**"!
{.is-warning}

> ***Atenção:*** Se não desejar habilitar algum parâmetro, como por exemplo o **"vacuum"**, basta não incluir esse parâmetro no arquivo **YAML**! Exemplo: simplesmente omita o parâmetro "**vacuum_config**"!
{.is-warning}



Aqui estão alguns conteudos para que você possa se **informar** mais sobre essas **configurações** e aprimorar seu **entendimento**, caso haja interesse:

- ➡️ [**Guide to Optimize Databricks, Spark and Delta Lake**](https://www.databricks.com/discover/pages/optimize-data-workloads-guide)

- ➡️ [**The need for optimize write on Apache Spark**](https://learn.microsoft.com/en-us/azure/synapse-analytics/spark/optimize-write-for-apache-spark)

- ➡️ [**Spark session property tweaks for better control**](https://www.linkedin.com/pulse/spark-session-property-tweaks-better-control-over-pathirippilly-mana/)

- ➡️ [**The Internals of Delta Lake**](https://books.japila.pl/delta-lake-internals/configuration-properties/#importbatchsizeschemainference)

# Serving Transformation

Na camada de Serving em transformação de dados refere-se à capacidade de fornecer dados de maneira ad-hoc ou sob demanda, possibilitando acesso imediato aos resultados transformados para consulta ou visualização. Ao habilitar esse parâmetro como True, irá permitir a visualização do dataset automaticamente no metabase. O dataset poderá ser encontrado no Metabase no seguinte schema e tabela:
Schema: <país_abreviado>_<domínio>_<subdomínio>
Tabela: <contexto**>_<nome_do_dataset>

## Configurações Serving Adhoc

Para concluir, se você precisar utilizar o Serving para inclusões Ad-hoc no Trino, será necessário aplicar a seguinte configuração também por meio do Sync!

>***Atenção:*** Em **ingestão**, não é necessário utilizar o **Ad-hoc**, uma vez que essa etapa não abrange a parte de *Serving*! Portanto, é crucial esclarecer que, para ingestão, não é preciso incluir a **configuração do Ad-hoc** no seu arquivo **YAML**.
{.is-warning}

***Dê uma olhada no exemplo abaixo.:***

```
...
datasets:
	...
	sync:
	 	adhoc:
     		active: True
	...
...

```

# Conclusão

Com a compreensão de todos os **modos** de **sincronização** e como o sincronismo opera, podemos concluir. Estamos à disposição para esclarecer quaisquer dúvidas :) . Abaixo, há um exemplo do arquivo YAML configurado em um pipeline!

```
dag:
  dag_type: 'postgres'
  dag_class: 'source'
  schedule: '0 11 * * *'
  system: 'Iris'
  country: 'Brazil'
  datasets:
    - name: 'completed_queries'
      active: True
      domain: 'TrinoDefault'
      entity: 'CompletedQueries'
      task_owner: 'Daniel Herbert Barbosa Junior'
      owner_team: 'Iris'
      database_type: 'postgresql'
      schema: 'public'
      columns: '*'
      sync:
        mode: "append_merge"
        vacuum_config:
          retain_hours: 0
        delta_config:
          custom_config: 
          	'spark.databricks.delta.properties.defaults.autoOptimize.autoCompact': 'True'
      connection_id: 'postgres_trino_default_eventlogger_ingestion_connection'
      metadata: !include ../metadata/trino_default_completed_queries.yaml
      notifier:
        type: 'ms_teams'
        receivers:
          - 'IrisTrinoWebhook'
```

## Dúvidas? 
Em caso de dúvidas entre em contato conosco através do nosso  <a href="https://dev.azure.com/AMBEV-SA/IRIS-Analytics%20Platform/_boards/board/t/AnalyticsTeam/Customer%20Service" target="_blank" >**Board de operações**</a>.
