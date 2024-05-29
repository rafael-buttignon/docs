# Migração de dados Históricos

## Objetivo

Atualmente, temos um processo para a migração de dados históricos que pode ser realizado seguindo regras específicas, as quais serão detalhadas abaixo. Estas regras são essenciais para garantir a integridade do seu Schema do dado. Por favor, leia atentamente os tópicos a seguir para compreender plenamente o procedimento de migração de dados históricos.

Antes de começarmos, é importante esclarecer que é necessário seguir um determinado fluxo. Porém, antes de avançar neste fluxo, vou detalhar as regras e cenários que você precisa compreender para realizar uma migração de dados eficaz.

Primeiramente, é possível migrar dados da arquitetura 1.0 para a 2.0, bem como da arquitetura 2.0 para a mesma versão 2.0. Existem diversas razões pelas quais essa migração pode ser necessária!

Abaixo, vamos falar do cenarios e dos passo a passos, que vamos precisar fazer para conseguir fazer a nossa migração!

## Cenarios

### <span style="color:Blue;">**Cenário Default**</span> - <span style="color:#333;">Conversão de um DataFrame para Delta <span>

O "cenário" zero, que mais apropriadamente não é um "cenário" é sim uma "especificação", requer que você identifique em que formato seus dados estão atualmente. Mas por que essa necessidade? Simples, na arquitetura 2.0, podemos ter dados no formato delta, enquanto na 1.0 eles podem estar em avro, parquet, entre outros. Para facilitar o processo de migração, é crucial especificar o formato dos dados que serão migrados. Isso ocorre porque nosso objetivo é sempre convertê-los para o formato delta, o **padrão** adotado na arquitetura 2.0.

***Imagem Exemplo:***
<img src="https://imgur.com/lkE3DnT.png" width="680" alt="Imgur">

### <span style="color:Blue;">**Cenário 01**</span> - <span style="color:#333;">Schemas, Particionamento & Data Types Iguais</span>
  
O primeiro cenário é o mais básico, no qual não há diferenças em schemas, particionamento e tipos de dados. Tudo é uniforme, o que simplifica a fusão de schemas. Na perspectiva da Iris, este cenário é ideal para migrações de 2.0 para 2.0. Porém, se esse cenário não se adequa à sua situação de migração, prossiga com a leitura dos próximos cenários!
  
***Imagem Exemplo:***  
<img src="https://imgur.com/77OKD8a.png" width="680" alt="Imgur">
  
### <span style="color:Blue;">**Cenário 02**</span> - <span style="color:#333;">Schema, Particionamento Iguais & Data Types Diferentes</span>
  
O segundo cenário caracteriza-se principalmente pela diferença nos tipos de dados (Data Types). Sob a perspectiva da Iris, este cenário é frequentemente encontrado em migrações de 2.0 para 2.0. Essas migrações ocorrem quando, por algum motivo, houve alterações no schema da fonte, tornando necessária uma mudança de path. E nesse caso o segundo cenário resolveria o problema.
  
***Imagem Exemplo:***  
<img src="https://imgur.com/MqioKoH.png" width="680" alt="Imgur">
  
### <span style="color:Blue;">**Cenário 03**</span> - <span style="color:#333;">Partições Diferentes, Data Types Diferentes & Schema Iguais</span>
  
O terceiro cenário é um dos mais abrangente e se aplica quando os dados a serem migrados possuem partições e tipos de dados distintos. Este é o cenário ideal para a migração nesse contexto. No entanto, ele não se aplica quando possui alterações de schemas.
  
***Imagem Exemplo:***  
<img src="https://imgur.com/aVOxkS5.png" width="680" alt="Imgur">
  
### <span style="color:Blue;">**Cenário 04**</span> - <span style="color:#333;">Schemas, Data Types & Partições Diferentess</span>
  
O quarto é ultimo cenário é o mais abrangente e engloba todos os aspectos discutidos até agora. Se seus dados possuem partições distintas, tipos de dados diversos e schemas diferentes, este é o cenário indicado. Contudo, uma observação importante é que, em relação à mudança de schemas, existem características que podem impactar tanto o schema quanto o dado em si. É recomendado entender esses detalhes observando a imagem a seguir.
  
***Imagem Exemplo:***  
<img src="https://imgur.com/qaaeDLD.png" width="680" alt="Imgur">
  
---
  
Depois de compreender e selecionar o cenário mais apropriado para sua migração de dados históricos, é crucial reconhecer que o schema base deve sempre ser o da arquitetura 2.0. Essa é a arquitetura que define a estrutura atual de ingestão. Não podemos alterar o Schema da 2.0, pois ele deve continuar recebendo dados do sistema de origem. Dado que a configuração foi estabelecida pelo próprio usuário ao criar a ingestão, essa integridade não deve ser comprometida.

O ponto-chave é que, especialmente quando consideramos o cenário quatro, que necessariamente pode envolver a remoção ou adição de colunas, o schema deve consistentemente seguir a estrutura predefinida na arquitetura 2.0. A predefinição que foi configurada pelo usuário com base nos dados recebidos da fonte de ingestão. Pois qualquer modificação na estrutura de dados configurada no yaml pode prejudicar o processo de ingestão. Portanto, essa compreensão é essencial para a equipe que deseja realizar a migração compreenda.
  
>**Para esclarecer:**  O schema padrão será sempre o da arquitetura 2.0. Independente de suas intenções, não é possível adotar um schema baseado na arquitetura 1.0, especialmente considerando a necessidade de consumir dados de uma fonte já especificada no metadata. Portanto, analise e compreenda a imagem acima para eliminar quaisquer dúvidas.
{.is-info}
  
### <span style="color:Red;">**Importante**</span>
Em todos os cenários mencionados, é altamente recomendável que os nomes das colunas em seu esquema sejam idênticos aos do esquema que você pretende migrar. Por exemplo, se na Arquitetura 1.0 você possui uma coluna com o nome 'id_completo', na Arquitetura 2.0 essa coluna deve ser mantida como 'id_completo', sem variações mínimas como 'idcompleto'. A falta de correspondência exata nos nomes das colunas pode resultar na incapacidade de unir esses dados, devido à diferença sutil no esquema. A prática correta é manter os nomes idênticos, tornando a criação de correspondências exatas a melhor abordagem sempre que possível. Isso garante a compatibilidade entre os esquemas e evita problemas de integração.
  
## Passo a Passo
  
### <span style="color:Blue;">**Migração de dados Históricos**</span> <span style="color:#333;">6 Passos</span>
  
Com o entendimento básico sobre o processo de migração e a escolha do cenário adequado, avancemos para o passo a passo! Iremos abordar desde a pausa da dag para a migração até a criação do card no board de atendimento do time da Iris. 
  
  
*Vamos relembrar a imagem acima, e seguir os passos a passo*. 
<img src="https://imgur.com/fZSiBtI.png" width="900" alt="Imgur">
  
**Passo [1] - Entender os cenários de migrações.**
  
Se você já compreendeu qual cenário selecionar para sua migração, ótimo. Caso contrário, releia a seção para garantir, pois essa informação será necessária ao especificar em seu card.
  
**Passo [2] - Pausar os processos que vão ser migrados**
  
**Arquitetura [1.0]**  
 - Antes de tudo, no seu processo da arquitetura 1.0, é necessário pausar ou desativar a operação que será migrada. Afinal, não é possível realizar a migração com um processo em andamento, correto? Assim, certifique-se de pausar ou desativar seu processo da arquitetura 1.0 antes da criação do card no board.
  
**Arquitetura [2.0]**    
  - Com o processo da arquitetura 1.0 pausado ou desligado, você pode pausar o processo da arquitetura 2.0 também bem fácil! O correto é pausar a task via pull request, e não pausar sua dag, com todas as outras tasks! O que você pode fazer é seguir o processo da imagem baixo!

***Imagem Exemplo:***  
<img src="https://imgur.com/BN00Im3.png" width="680" alt="Imgur">
  
Para pausar a task, defina o valor de 'active' como **False** na task que deseja migrar. Um lembrete crucial para processos que envolvem **filas**: monitore para garantir que as filas não atinjam seu limite máximo. Esse alerta é particularmente relevante para filas, já que alguns conectores, como o **RabbitMQ**, possuem um limite definido para armazenamento de mensagens. No entanto, se você não estiver pausando um processo relacionado a uma fila, não há necessidade de preocupações adicionais.
  
Após definir o active da sua task, para **False**, você pode gerar sua dag com o comando ***pyiris ingestion --generate-dag***, e criar seu Pull Request, para que sua task seja pausada!
  
**Passo [3] - Criar card de migração no board de operações**
  
Com os passos anteriores feitos com sucesso, vamos para a nossa criação do card no board de atendimento do time da Iris! Para acessar o board, 🧑‍💻 [**Clique Aqui!**](https://dev.azure.com/AMBEV-SA/IRIS-AnalyticsPlatform/_boards/board/t/AnalyticsTeam/Customer%20Service)

Acessando o board, você deve criar o card de Migração de dados históricos, como mostra na imagem abaixo.
  
***Imagem Exemplo:***  
<img src="https://imgur.com/l7VwSFZ.png" width="680" alt="Imgur">
  
Após clicar em **'Adicionar ao topo (add to top)'**, será necessário preencher algumas informações.

***Imagem Exemplo:***  
<img src="https://imgur.com/S4WwBKf.png" width="680" alt="Imgur">
  
Agora é importante preencher todos os campos conforme indicado na imagem acima, nos campos que estão destacados em vermelho!
  
É necessário preencher o campo do ***"Título"*** com uma descrição que destaque a migração da arquitetura atual para a nova arquitetura necessária. Certifique-se de ressaltar claramente a transição entre as arquiteturas.

Na seção de ***"Descrição"***, é crucial incluir observações e atenções, caso sejam necessários. É fundamental fornecer uma breve explicação sobre o processo que vai ser migrado.
  
Agora, dentro da seção ***"Custom"***, continue preenchendo determinados campos. No exemplo do ***"Campo 01"***, é essencial indicar se você desligou ou pausou os processos da Arquitetura 1.0. Caso a migração não envolva a Arquitetura 1.0, você pode manter a seleção como **Sim**.
  
No ***"Campo 02"***, é necessário indicar se você pausou o processo da Arquitetura 2.0. Se possível, forneça o link direto para o Pull Request, de modo que ele possa ser revisado e mergeado antes de a equipe de operações atuar no seu card.
  
Agora, nos campos ***"03 e 04"***, é crucial indicar o formato atual dos dados que serão migrados. É importante notar que estamos nos referindo ao formato dos dados a **serem migrados**, não aos dados já presentes na Arquitetura 2.0. No ***"Campo 04"***, você deve especificar o cenário de migração que pretende utilizar, de acordo com as opções apresentadas no início dessa documentação.
  
***Imagem Exemplo:***  
<img src="https://imgur.com/7cjYOx2.png" width="680" alt="Imgur">
  
Na imagem acima, você observa que devemos preencher o ***"Campo 05"***. Aqui, você deve inserir com precisão o caminho do antigo dado que será migrado, seja o caminho da Arquitetura 1.0 ou 2.0, que será então migrado para a Arquitetura 2.0.
  
No próximo ***"Campo 06"***, é necessário inserir o caminho atual de onde esses dados estão localizados na Arquitetura 2.0. É fundamental compreender que este campo deve ser preenchido com o caminho atual da Arquitetura 2.0, uma vez que esse caminho servirá como base para garantir a continuidade da ingestão de forma precisa.
  
No último ***"Campo 07"***, é crucial incorporar o conceito de **'merge_key'**, que desempenha um papel fundamental. O **'merge_key'** representa essencialmente a chave compartilhada entre os dois conjuntos de **dados ('dataframes')**. Essa chave é fundamental para permitir a fusão precisa dos dados, com base no cenário de sua escolha. Em suma, o **'merge_key'** é um campo comum presente em ambos os conjuntos de dados, que é selecionado como a chave principal para facilitar a fusão entre eles. Importante, você entender é verificar seus dados, para apresentar a melhor escolha de **'merge_key'**.
  
***Imagem Exemplo:***  
<img src="https://imgur.com/e5Z7Rbp.png" width="330" alt="Imgur">
  
Após criar o seu Card, é importante assiná-lo com o seu nome e aguardar a sua vez de atendimento. Reconheço que esse procedimento pode ser mais complexo, o que pode resultar em um maior tempo de espera. No entanto, para evitar muito tempo de espera, sugiro que você comunique a alguém da equipe **Iris**, preferencialmente da equipe de operações, sobre a sua necessidade urgente de migração desse processo. Destaque que não é viável manter o processo pausado por muito tempo. Assim a equipe de operações a atender a sua solicitação o mais rápido possível.
  
**Passo [4] - Validar os dados que foram migrados**
  
Agora, uma vez que a equipe de operações tenha agido em seu card e concluído a tarefa de migração, você terá a oportunidade de validar os dados migrados. É altamente recomendado realizar uma validação minuciosa, a fim de evitar possíveis problemas futuros. A responsabilidade total por essa validação recai sobre você e sua equipe. Como o proprietário dos dados, você decide como essa validação será conduzida. Isso pode envolver análise de questões relacionadas aos dados, schemas, tipos de dados, integridade, correlação dos dados, entre outros...
  
Não adie essa etapa importante de validação. Ela é crucial para manter o fluxo contínuo do seu pipeline de ingestão. Esta etapa não depende da equipe Iris; é exclusivamente da responsabilidade do proprietário dos dados (você). Para conduzir a validação dos dados, você pode utilizar um dos ambientes disponíveis na infraestrutura da Iris.
  
**Passo [5] - Despausar o processo que foi pausado para a migração! Para continuar o fluxo de ingestão**
  
Após concluir a validação dos dados, é importante reativar o processo da Arquitetura 2.0. Você pode realizar essa etapa seguindo os mesmos passos descritos na **Etapa [2]**. Para reativar o pipeline, basta definir o valor de **'active'** como **True**, gerar a DAG correspondente e enviar o Pull Request para o repositório. E aguardar a aprovação por parte da equipe Iris antes de prosseguir.
  
**Passo [6] - Validou os dados? Pipeline rodou? Informar no card a deleção dos dados históricos antigos.**
  
Assim que despausar, rode a dag. Você pode esperar até a execução ou ir no proprio Airflow, e clicar em **'trigger'**, para ter a execução instantânea.
  
***Imagem Exemplo:***  
<img src="https://imgur.com/K9WUAD0.png" width="1080" alt="Imgur">
  
Agora é hora de analisar o seu pipeline para determinar se ocorrerão erros ou se ele será executado com êxito. Se o pipeline for bem-sucedido, isso confirmará que a migração dos dados foi realizada corretamente, de acordo com os dados especificados pelo Schema da Arquitetura 2.0. No entanto, caso ocorra uma falha que não existia anteriormente, é importante investigar. Essa falha possivelmente indica um problema no Schema dos dados e sugere que a migração pode não ter sido executada como esperado. Portanto, é fundamental realizar uma validação detalhada para evitar erros durante a continuação do processo de ingestão dos dados.
  
Nos melhores do mundo, uma vez que o seu pipeline seja executado com sucesso, é altamente recomendado realizar uma validação adicional dos dados. Isso garantirá que você tenha total confiança de que os dados que foram migrados estão de fato presentes e corretamente formatados, de acordo com suas expectativas. Caso tudo esteja em ordem, você pode prosseguir para o card e informar para o time de operações que os caminhos dos dados históricos antigos, que não serão mais utilizados, podem ser excluídos.
  
***Imagem Exemplo:***  
<img src="https://imgur.com/pw1685u.png" width="680" alt="Imgur">
  
Aqui está um exemplo do comentário que deixei no Card, no qual instruí a equipe de operações a proceder com a exclusão dos seguintes caminhos que não serão mais utilizados.
  
>**Importante**: É crucial garantir que você tenha absoluta certeza sobre quais caminhos está solicitando à equipe de operações para deletar. Isso se deve ao fato de que, uma vez realizada a exclusão, não será possível recuperar os dados. Portanto, é essencial estar totalmente seguro antes de prosseguir.
{.is-warning}
  
É de suma importância destacar que é necessário comunicar à equipe de operações os caminhos que serão deletados. Isso se deve ao fato de que não podemos arcar com custos associados a ambas as arquiteturas simultaneamente.
  
## FAQ
### <span style="color:Blue;">**Finalização**</span> - <span style="color:#333;">Vídeo Tutorial<span>
  
Após revisar toda a documentação, você ainda tem alguma dúvida sobre os procedimentos ou cenários abordados? Para auxiliar na compreensão dos passos necessários e do processo em geral, foram preparados alguns vídeos explicativos. Estes vídeos fornecem uma visão ampla da migração e orientações sobre como proceder. Se as dúvidas persistirem, recomendamos assistir aos tutoriais em vídeo disponibilizados abaixo. Estamos à disposição para esclarecer qualquer questionamento adicional.
  
<br>

<iframe src="https://anheuserbuschinbev.sharepoint.com/sites/irisplatform/_layouts/15/embed.aspx?UniqueId=30780d21-b017-458f-8c36-aececc0a2a16&embed=%7B%22ust%22%3Atrue%2C%22hv%22%3A%22CopyEmbedCode%22%7D&referrer=StreamWebApp&referrerScenario=EmbedDialog.Create" width="640" height="360" frameborder="0" scrolling="no" allowfullscreen title="Entendendo Migracao.mp4"></iframe>


