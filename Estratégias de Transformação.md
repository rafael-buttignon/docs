
Hoje, temos a ingestão de dados com algumas funcionalidades adicionais para melhorar sua utilização. Isso proporciona benefícios significativos ao usuário, por meio de parâmetros e funcionalidades adequadas. Com isso em mente, surge o benefício da estratégia de transformação, onde podemos distinguir dois tipos de estratégias.

# Transformações Defaults

Hoje, as transformações defaults ocorrem em todas as ingestões, e as transformações defaults já são aplicadas na primeira camada, conhecida como 'rawzone'. As transformações defaults aplicadas são as seguintes:

## <span style="color:#333;">Aplicações Defaults na camada <span style="color:Red;">**RawZone**</span></span>:

- **Padrão snake_case**
  - São aplicados nomes em snake_case em todas as colunas do dataframe. Portanto, se você passar um esquema sem o padrão de snake_case, o método padrão aplicará a regra de snake_case para todo o dataframe em si, colocando todas as colunas no padrão snake case.
  
- **Padrão Ingestion Date Partition**
	- Assim que você faz a ingestão de um dado, ocorrerá essa transformação por padrão, que consiste na adição das colunas **[year, month, day]** ao seu dataset. Caso você não escolha uma partição específica para o seu dataframe, ele usará automaticamente as novas colunas adicionadas para particionar, ou seja, **[year, month, day]**.
  - Entretanto, é possível adotar uma estratégia diferente e **particionar** por uma coluna selecionada pelo usuário. Uma vez escolhida uma coluna diferente para o particionamento, ela será utilizada em ambas as camadas **RawZone/TrustedZone**. É importante ressaltar que o nome da coluna deve ser passado de acordo com a grafia original do dado de origem, **exemplo.:**
  ~~~yaml
  datasets:
    partition_by: "id"
  ~~~
  Caso você não passe nenhuma opção de **"partition_by"**, será utilizado o valor padrão, que consiste na data atual com as colunas **[year, month, day]**, para ambas as camadas.

<br>

# Transformações Opcionais

Agora vamos falar sobre as transformações opcionais. Se você deseja utilizá-las, deve aplicá-las em seu arquivo YAML para que funcionem corretamente! Lembrando que essas transformações opcionais são uma forma de ajudá-lo com tarefas repetitivas do seu dia a dia.

As transformações opcionais ocorrerão apenas na **trustedzone**. A rawzone é uma camada de acesso restrito aos usuários, servindo apenas como backup de emergência para a equipe da plataforma. Dessa forma, os dados nessa camada devem ser preservados da maneira mais fiel possível às informações existentes nas fontes.

## <span style="color:#333;">Aplicações Opcionais na camada <span style="color:Blue;">**TrustedZone**</span></span>:

- **Padrão Drop Duplicate**
  - Outra transformação padrão que ocorre é a transformação de remoção de duplicatas (drop duplicates), porém essa transformação ocorre na camada da trustedzone. Basicamente, você pode escolher entre dois tipos, **"based_on_columns"** e **"exclude_only"**:
  
  ---
  
  **based_on_columns**: Opção de exclusão de duplicados baseado apenas em colunas **selecionadas**.
  ~~~yaml
  datasets:
    transformations:
      drop_duplicate:
        based_on_columns: ['col1', 'col2', 'col3']
  ~~~
  **exclude_only**: Opção de exclusão de duplicados considerando todas as colunas do dataset, com excessão das "exclude_only".
  ~~~yaml
  datasets:
    transformations:
      drop_duplicate:
        exclude_only: ['col1', 'col2', 'col3']
  ~~~
  Portanto, existem essas opções que você pode passar via **YAML**, como nos exemplos acima. Cada uma segue um algoritmo que você decide escolher, então escolha corretamente. 
  
 	<br>Recapitulando, o "**based_on_columns**" são as colunas que você deseja comparar. Por exemplo, se eu quero que o meu método "**based_on_columns**" compare **['ID', 'Name']**, basicamente ele irá comparar esses dois campos do dataframe.<br>

  </br>Já o "**exclude_only**" irá comparar as colunas que não estão presentes na lista "**exclude_only**". Por exemplo, se eu não quero comparar **['ID', 'Name']**, o algoritmo irá apenas comparar outras colunas do dataset.
  
  </br>Há ainda a opção de não indicar nenhuma das duas opções acima. Nesse caso, a estratégia será a utilização de **todas as colunas no momento da comparação de duplicados**. Basicamente, você não precisará passar nada no YAML, ele irá entender que você não indicou nada e vai utilizar a estratégia padrão...
  
  </br>Basicamente, ele irá comparar todas as colunas e linhas do dataframe. Se houver alguma duplicata com todas as colunas e linhas iguais, ele removerá e manterá apenas uma, como no exemplo abaixo:
  ~~~yaml
  +---+-----+--------+---------+----+-----+---+
  | id|value|quantity|quantity2|year|month|day|
  +---+-----+--------+---------+----+-----+---+
  | 1| 200|       5|       1500|2023|    7|  6|
  | 1| 200|       5|       1500|2023|    7|  6|
  | 1| 200|       5|       1500|2023|    7|  6|
  +---+-----+--------+---------+----+-----+---+
  ~~~
  **Resultado:**
  
  ~~~yaml
  +---+-----+--------+---------+----+-----+---+
  | id|value|quantity|quantity2|year|month|day|
  +---+-----+--------+---------+----+-----+---+
  | 1| 200|       5|       1500|2023|    7|  6|
  +---+-----+--------+---------+----+-----+---+
  ~~~
  ---
***Imagem Exemplo:***
<img src="https://imgur.com/390bWly.png" width="1080" alt="Imgur">

---

- **Hash Columns**
  - A transformação hash faz com que você converta uma coluna em funções hash de 256 bits. Portanto, se você deseja aplicar essa transformação a uma coluna do seu dataframe, basta utilizar o seguinte exemplo:
	  ~~~yaml
     datasets:
        transformations:
          hash:
            from_columns: ["name", "CPF"]
  	~~~
***Imagem Exemplo:***
<img src="https://imgur.com/tGVPHoA.png" width="1080" alt="Imgur">

---

- **Cleaning Transformation**
  - Basicamente, o método de **cleaning** se refere a uma técnica de limpeza de dados que pode ser utilizada de três maneiras, juntamente com uma expressão regular de sua preferência. Na transformação do método cleaning, você pode utilizar das seguintes maneiras: **"apply_all_columns",** **"based_on_columns"** ou **"exclude_only"**, além de passar o **"cleanup_regex"** como sua expressão regular para a limpeza dos dados. Caso não queira usar a expressão regular que já vem por padrão, ela será mostrada nos exemplos abaixo.:
  
  ---
  
  **apply_all_columns**: Basicamente, a estratégia **"apply_all_columns"** consiste em aplicar a limpeza para todas as colunas, juntamente com uma expressão regular de sua escolha na variável **"cleanup_regex"**. É importante lembrar que é sempre necessário testar o regex que você criou para evitar transformações incorretas. A responsabilidade total recai sobre o usuário que utilizou a transformação e criou o regex.
  
  </br><span style="color:Red;">**Atenção**</span>: É importante utilizar o apóstrofo **(' ')** na variável **"cleanup_regex"** em vez de utilizar aspas duplas **(" ")**.
  
  ~~~yaml
  datasets:
    transformations:
      cleansing:
        apply_all_columns: True
        cleanup_regex: '[\\\n\t\|/*\r;]|@$'
  ~~~
    
	<br>**based_on_columns**: Opção de limpeza baseado apenas em colunas **selecionadas**:

  ~~~yaml
  datasets:
    transformations:
      cleansing:
        based_on_columns: ['col1', 'col2', 'col3']
        cleanup_regex: '[\\\n\t\|/*\r;]|@$'
  ~~~
  
  <br>**exclude_only**: Opção de limpeza dos dados considerando todas as colunas do dataset, com excessão das "exclude_only".

  ~~~yaml
  datasets:
    transformations:
      cleansing:
        exclude_only: ['col1', 'col2', 'col3']
        cleanup_regex: '[\\\n\t\|/*\r;]|@$'
  ~~~
  
  Portanto, o algoritmo de cleansing para os tipos **"based_on_columns"** e **"exclude_only"** segue o mesmo exemplo mencionado anteriormente sobre **"drop_duplicates"**. Se você já leu o exemplo anterior sobre a remoção de duplicatas, não precisa se preocupar em lê-lo novamente.

 	<br>Mas recapitulando novamente, o "**based_on_columns**" são as colunas que você deseja aplicar o metodo de limpeza. Por exemplo, se eu quero que o meu método "**based_on_columns**" faça limpeza em **['ID', 'Name']**, basicamente ele irá limpar os dados desses dois campos do dataframe.<br>

  </br>Já o "**exclude_only**" irá limpar as colunas que não estão presentes na lista "**exclude_only**". Por exemplo, se eu não quero fazer a limpeza dos dados dos campos **['ID', 'Name']**, o algoritmo irá apenas utilizar o metodo de limpeza nas outras colunas do dataset.
  
  </br>Além disso, você pode utilizar o regex padrão que deixamos pré-configurado, caso não queira criar um regex completamente do zero. Para utilizar o regex padrão, basta **não** passar a variável **"cleanup_regex"**.
  
  </br>Você pode utilizar dessa maneira tanto para o **"apply_all_columns"**, **"based_on_columns"** ou **"exclude_only"**.
    ~~~yaml
  datasets:
    transformations:
      cleansing:
        apply_all_columns: True
  ~~~
  
  </br>Basicamente, o regex **pré-configurado** que deixamos como padrão é a seguinte expressão regular: "**[\\n\t|/]|@$**". Se ela atender às suas regras de limpeza, você pode seguir os exemplos acima sem precisar criar um regex do zero. No entanto, se preferir criar seu próprio regex por questões proprias, fique totalmente à vontade.
  
  </br><span style="color:brown;">**O que esse regex pré-configurado faz?**</span>: Esse regex é uma expressão regular que corresponde a certos caracteres especiais em uma string. Juntando tudo, a expressão regular "**[\\\n\t\|/]|\@$**" corresponde a qualquer um dos caracteres \ (barra invertida), n (nova linha), t (tabulação), | (pipe), / (barra), @ (arroba) ou $ (dólar) em uma string.
  
  </br>Então, basicamente, o regex pré-configurado acima remove todas as características definidas nos seus dados, caso eles as contenham. No entanto, é importante ressaltar que cada caso é único, e é fundamental que você conheça o regex que está utilizando. Portanto, é sempre recomendável criar o seu próprio regex caso seus dados possuam regras diferentes das mencionadas.
  
  </br>***Imagem Exemplo:***
<img src="https://imgur.com/XjyioP0.png" width="1080" alt="Imgur">

<br>

---
- **Checkpoint Transformation**
  - O método "checkpoint" é um recurso que permite a criação de uma coluna adicional em seu DataFrame. Essa coluna serve como uma espécie de ponto de controle para monitorar a entrada de dados. Na prática, o método gera uma coluna de data, hora, minutos e segundos **"timestamp"** em seu DataFrame. Para utilizá-lo, você deve fornecer o parâmetro como **"True"**.

  - A utilidade do método **"checkpoint"** é evidente quando há necessidade de um rastreamento aprimorado dos dados ou quando é preciso validar uma determinada regra. Ele se torna extremamente útil, especialmente se a fonte de dados que está sendo ingerida **não** oferece essa funcionalidade. Portanto, você pode empregar este método como uma ferramenta auxiliar no rastreamento e validação de dados.
  
  ~~~yaml
  datasets:
    transformations:
      checkpoint: True
  ~~~
  
O Checkpoint permite adicionar uma coluna ao seu dataframe com um timestamp formatado como string. Esta coluna reflete a data e hora exatas em que o dataframe foi salvo no lake. O formato utilizado é **"%d-%m-%Y %H:%M:%S"**, resultando em uma saída semelhante a **"2023-08-02 15:30:45"**.

> É **crucial** lembrar que o checkpoint adiciona uma nova coluna ao seu dataframe, então não há necessidade de **incluí-la no metadata**. No entanto, se seus dados já **estiverem no lake** sem a transformação do checkpoint e você desejar adicioná-la posteriormente, será necessário utilizar o Delta para **sobrescrever** seus dados ou esquema, permitindo a criação dessa nova coluna. Para saber mais sobre delta, [**Clique Aqui!**](/iris-ingestion/deep-dive/data-anti-corruption)
{.is-warning}

  
</br>***Imagem Exemplo:***
<img src="https://imgur.com/3xr24y9.png" width="1080" alt="Imgur">

## Aplicando mais de uma Transformação

No entanto, você deve estar se perguntando: ***"Será que posso utilizar a transformação de cleansing e também a de hash? Ou posso utilizar múltiplas transformações?"*** E a resposta é sim, você pode utilizar quantas transformações desejar. Basta escrever quais transformações você quer no seu arquivo YAML, como mostrarei no exemplo abaixo.

### <span style="color:black;">Aplicando Múltiplas Transformações</span>:

  ~~~yaml
  datasets:
    transformations:
      cleansing:
        apply_all_columns: True
        cleanup_regex: '[\\\n\t\|/*\r;]|@$'
      hash:
      	from_columns: ['cpf']
      checkpoint: True
  ~~~

Por exemplo, no exemplo acima, estou aplicando duas transformações no mesmo arquivo YAML de uma task. Basicamente, estou aplicando a transformação de limpeza e também a transformação de hash para uma coluna específica na qual desejo fazer o hash.

Você pode aplicar quantas transformações desejar. Basicamente, basta seguir as regras descritas em cada passo, aplicando qualquer regra mencionada neste artigo.

# Conclusão

É importante ressaltar que você pode utilizar essas transformações para facilitar o seu trabalho no dia a dia, caso algum cenário de negócio se enquadre nessas transformações. É crucial avaliar com cautela a transformação que você aplicará nos seus dados, para evitar perda de informações ou alterações indesejadas na estrutura dos dados. Portanto, se você avaliar esses requisitos, fica claro que utilizar as transformações apresentadas neste artigo trará diversos benefícios, como economia de tempo, melhoria na qualidade dos dados e utilidade adicional. Espero que aproveite essas funcionalidades e, caso tenha alguma dúvida, a equipe da Iris está disponível para responder.