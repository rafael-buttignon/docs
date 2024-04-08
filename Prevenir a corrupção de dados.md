# Prevenir a corrupção de dados - Aplicação e Evolução do Schema

Os dados, como nossas experiências, estão sempre evoluindo e se acumulando. Para acompanhar, nossos modelos mentais do mundo devem se adaptar a novos dados, alguns dos quais contêm novas dimensões, novas maneiras de ver as coisas que antes não tínhamos concepção. Esses modelos mentais não são diferentes do schema de uma tabela, definindo como categorizamos e processamos novas informações.

Isso nos leva ao gerenciamento de schema. À medida que os problemas e requisitos de negócios evoluem ao longo do tempo, a estrutura de seus dados também evolui. Com o <kbd>Delta Lake</kbd>, à medida que os dados mudam, é fácil incorporar novas dimensões. Os usuários têm acesso a uma semântica simples para controlar o schema de suas tabelas. Essas ferramentas incluem a imposição de schema, que impede que os usuários poluam acidentalmente suas tabelas com erros ou dados inúteis, bem como a evolução do schema, que permite adicionar automaticamente novas colunas de dados avançados quando essas colunas pertencem.
## O que é aplicação de Schema?

A imposição de schema, também conhecida como validação de schema, é uma proteção no <kbd>Delta Lake</kbd> que garante a qualidade dos dados ao rejeitar gravações em uma tabela que não corresponde ao schema da tabela. Assim como o gerente da recepção de um restaurante movimentado que só aceita reservas, ele verifica se cada coluna nos dados inseridos na tabela está em sua lista de colunas esperadas (ou seja, se cada uma tem uma “reserva”) e rejeita todas as gravações com colunas que não estão na lista.

## Como funciona a aplicação do Schema?

O Delta Lake usa validação de schema em <kbd>write</kbd> , o que significa que todas as novas gravações em uma tabela são verificadas quanto à compatibilidade com o schema da tabela de destino no momento da gravação. Se o schema não for compatível, o Delta Lake <kbd>cancela a transação</kbd> completamente (nenhum dado é gravado) e gera uma exceção para informar o usuário sobre a incompatibilidade.

Para determinar se uma gravação em uma tabela é compatível, o Delta Lake usa as seguintes regras:

- **Não pode conter nenhuma coluna adicional que não esteja presente no schema da tabela de destino**. Por outro lado, não há problema se os dados de entrada não contiverem todas as colunas da tabela, essas colunas serão simplesmente atribuídas a valores nulos.

- **Não pode ter tipos de dados de coluna diferentes dos tipos de dados de coluna na tabela de destino**. Se a coluna de uma tabela de destino contiver dados StringType, mas a coluna correspondente no DataFrame contiver dados IntegerType, a imposição do schema gerará uma exceção e impedirá que a operação de gravação ocorra.

- **Não pode conter nomes de coluna que diferem apenas por maiúsculas e minúsculas.** Isso significa que você não pode ter colunas como '**Foo'** e **'foo'** definidas na mesma tabela. Embora o Spark possa ser usado no modo sensível a maiúsculas ou minúsculas (padrão), o Delta Lake preserva maiúsculas e minúsculas, mas não diferencia maiúsculas de minúsculas ao armazenar o schema. Parquet diferencia maiúsculas de minúsculas ao armazenar e retornar informações de coluna. Para evitar possíveis erros, corrupção de dados ou problemas de perda (que experimentamos pessoalmente na Databricks), decidimos adicionar essa restrição.

## O que é a evolução do Schema?

A evolução do schema é um recurso que permite que os usuários alterem facilmente o schema atual de uma tabela para acomodar dados que estão mudando ao longo do tempo. Mais comumente, é usado ao executar uma operação de <kbd>acréscimo ou substituição</kbd>, para adaptar automaticamente o schema para incluir uma ou mais novas colunas.

## Como funciona a evolução do Schema?
Seguindo o exemplo da seção anterior, os desenvolvedores podem facilmente usar a evolução do schema para adicionar as novas colunas que foram rejeitadas anteriormente devido a uma incompatibilidade de schema ou mudanças de tipos, nomes, etc.. A evolução do schema é ativada adicionando <kbd>**mergeSchema**</kbd> ou <kbd>**overwriteSchema**</kbd> no yaml de sua Dag.

## Qual tipo de evolução de Schema, devo utilizar?

Aprenda a diferenciar os tipos de evolução de schema, antes mesmo de sair fazendo modificações no seu pipeline.

### Como utilizar MergeSchema?

<kbd>**MergeSchema**</kbd> é o caso mais padrão, que é utilizado nos seguintes cenários:

- <kbd>1 Cenário:</kbd>
	-	Criação de uma <kbd>nova coluna </kbd>
- <kbd>2 Cenário:</kbd>
	- Mudança do tipo de dado de <kbd>non-nullable para nullable</kbd>
- <kbd>3 Cenário:</kbd>
	- Upcasts from <kbd>ByteType -> ShortType -> IntegerType</kbd>

O **mergeSchema**, você apenas faz <kbd>**append**</kbd>, então é o modo **mais** seguro de se utilizar e que não precisa fazer a exclusão do seu diretorio.

Básicamente o melhor cenário de se utilizar o **mergeSchema**, é quando você precisa <kbd>incluir uma nova coluna</kbd>. Essa nova coluna quando é adicionada, passa os dados como **nulos** aonde **não possui aquele dado em específico**. 

**Exemplo**:

~~~
sync_mode: 'incremental_append'
~~~

![Imgur](https://imgur.com/ACgapXS.png)

> **Informação**: Aplicando apenas essa linha de código em sua task, já estará funcionando a evolução de schema!
{.is-info}

Lembre-se utilize apenas se enfrentou algum **problema** relatado acima! Essas opções, podem influenciar diretamente sem seu dataset e em possíveis alterações inesperadas.

### Como utilizar o OverwriteSchema?

**OverwriteSchema** abordará algumas limitações do mergeSchema, como a necessidade de considerar a alteração dos tipos de dados nas mesmas colunas (por exemplo: <kbd>String para Integer</kbd>). Nesse cenário, todos os arquivos de dados do parquet precisariam ser gravados. OverwriteSchema pode ser responsável por **descartar** uma coluna, alterar o **tipo** de dados da coluna existente e/ou **renomear** nomes de colunas que diferem apenas por **maiúsculas** e **minúsculas**.

- <kbd>1 Cenário:</kbd> 
	- Mudanças de Tipo, Exemplo: <kbd>(IntegerType para StringType)</kbd>
- <kbd>2 Cenário:</kbd> 
	- Mudança de nome da coluna maiúsculas e minúsculas, Exemplo: <kbd>(Foo para foo)</kbd>
- <kbd>3 Cenário:</kbd>
	- Remover coluna
- <kbd>4 Cenário:</kbd> 
	- Alterar coluna

O **overwriteSchema**, você apenas faz **overwrite**, então e um pouco mais especifico, porque você vai reescrever a tabela para fazer alterações de schema **incompatíveis**.

Basicamente o melhor cenário de se utilizar o **overwriteSchema**, é quando você precisa alterar o tipo do dado da coluna, por exemplo: <kbd>(String para Integer)</kbd>, mas existem diversos cenários aonde o **overwriteSchema** pode se aplicar como falado acima. 

**Exemplo**:

~~~
sync_mode: 'overwrite'
~~~

![Imgur](https://i.imgur.com/y3ZsOXa.png)

> **Informação**: Aplicando apenas essa linha de código em sua task, já estará funcionando a evolução de schema!
{.is-info}


> **Atenção:** Você utilizando a opção **overwriteSchema**, você está "sobrescrevendo" seus dados. Léia o proximo bloco para entender melhor!
{.is-warning}

## Como a evolução do Schema é útil? E Porque Substituir o conteúdo ou Schema de uma tabela.

A evolução do schema pode ser usada sempre que você pretende alterar o schema da sua tabela (ao contrário de onde você acidentalmente adicionou colunas ao seu DataFrame que não deveriam estar lá). É a maneira mais fácil de migrar seu schema porque adiciona automaticamente os nomes de coluna e tipos de dados corretos, sem precisar declará-los explicitamente.

Às vezes você pode querer substituir uma tabela. Por exemplo:

- Você descobre que os dados na tabela estão <kbd>incorretos</kbd> e deseja substituir o conteúdo.
- Você deseja reescrever a tabela inteira para fazer alterações de schema incompatíveis (como alterar os tipos de coluna).

Embora você possa excluir todo o diretório de uma tabela Delta e criar uma nova tabela no mesmo caminho, isso não é recomendado porque:

- A <kbd>exclusão</kbd> de um diretório não é eficiente. Um diretório contendo arquivos muito grandes pode levar horas ou até dias para ser excluído.

- Você perde todo o conteúdo dos arquivos excluídos; é difícil recuperar se você excluir a tabela errada.

- A exclusão do diretório não é **atômica**. Enquanto você está excluindo a tabela, uma consulta simultânea lendo a tabela pode falhar ou ver uma tabela parcial.

Por isso no caso acima, se você quiser alterar o schema da tabela, você poderá substituir toda a tabela atomicamente utilizando o método **overwriteSchema**.

Existem vários benefícios com esta abordagem:

- **Substituir** uma tabela é muito mais **rápido** porque não precisa listar o diretório recursivamente ou excluir nenhum arquivo.

- A versão antiga da tabela ainda existe. Se você excluir a tabela errada, poderá recuperar facilmente os dados antigos usando Viagem no tempo. *(No caso da Viagem no Tempo é uma feature do Delta Lake, não implementada ainda!)*

- Devido às garantias de transação ACID do Delta Lake, se a substituição da tabela falhar, a tabela estará em seu estado anterior.

Além desses benefícios citados, você não precisara ficar abrindo card no board para pedir **deleção** dos dados antigos. O fluxo ocorrerá com mais facilidade. 

## Guia de Boas Práticas

Bom para finalizarmos, após você ter entendido como funciona a evolução de schemas, vamos para o guia de boas práticas!

No cenário ideal, sempre que você for incluir uma das duas opções **mergeSchema** ou **overwriteSchema**, você tem que entender que após a inclusão, você removera a <kbd>*"camada de prevenção"*</kbd>, no caso, a ingestão de sua tabela ficara **vulnerável** para possíveis erros. Então se você acidentalmente adicionar colunas, alterar tipos ou renomear a coluna, o Schema do seu DataFrame aplicara essas alterações, pois a opção de **mergeSchema** ou **overwriteSchema**, está habilitada, então ele ira aplicar sem nenhum aviso prévio é também á possibilidades de corromper sua tabela é bastante alta!

Nesse caso para mitigar efeitos é diminuir os riscos de corromper o dado com erros acidentalmente, o que recomendamos é após o uso do **mergeSchema** ou **overwriteSchema** o recomendado seria, que em outra ingestão ou em outra interação dentro do repositório, seja removido as opções de **mergeSchema** e **overwriteSchema** do yaml. Para que não ocorra possíveis erros. É também, para nós deixar sempre alerta de possíveis alterações e evoluções de Schemas. 

## Resumo

A imposição de schema rejeita quaisquer novas colunas ou outras alterações de schema que não sejam compatíveis com sua tabela. Ao definir e manter esses altos padrões, analistas e engenheiros podem confiar que seus dados têm os mais altos níveis de integridade e raciocinar sobre isso com clareza, permitindo que eles tomem melhores decisões de negócios.

Por outro lado, a evolução do schema complementa a aplicação, facilitando que as alterações pretendidas do schema ocorram automaticamente. Afinal, não deve ser difícil adicionar uma coluna, mudar tipo, e renomeá-los.

---

**Agora que você entendeu como funciona a prevenção a corrupção de dados e aplicação e evolução do schema vamos seguir para os próximos passos:**

- [:building_construction: **Construir Dags Mais Eficientes**](/iris-ingestion/deep-dive/dag-pools)