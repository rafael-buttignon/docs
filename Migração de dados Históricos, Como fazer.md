# Migração de dados Históricos

Atualmente, temos um processo para a migração de dados históricos que pode ser realizado seguindo regras específicas, as quais serão detalhadas abaixo. Estas regras são essenciais para garantir a integridade do seu Schema do dado. Por favor, leia atentamente os tópicos a seguir para compreender plenamente o procedimento de migração de dados históricos.

Para tornar as coisas mais claras e diretas, criei um vídeo tutorial que aborda a migração de dados. No entanto, é fundamental que você realize esse processo com extrema cautela, a fim de evitar qualquer impacto nos dados dos usuários, uma vez que se trata de um procedimento crítico.

Aqui estão os links para os processos que você precisará utilizar durante a migração. É **crucial** lembrar de verificar a disponibilidade do mount no ambiente onde a migração será realizada. Além disso, é essencial executar esses processos nas camadas da **rawzone** e **trustedzone** na arquitetura 2.0.

---
***Documentação***: leia a documentação **disponível para os usuários** para entender os cenários e entender o processo de migração de dados históricos.

- ➡️ [**Migração de dados Históricos**](https://iris.ambev.com.br/pt-br/data-ingestion/arq2-0/how-to/migracao-dados-historicos)

---

***Notebooks***: para realizar a migração;

- ➡️ [**Notebook nonprod-iris-intelligence**](https://adb-5055431857639398.18.azuredatabricks.net/?o=5055431857639398#folder/3188030616045265)
- ➡️ [**Notebook prod-iris-intelligence**](https://adb-8362622574829379.19.azuredatabricks.net/browse/folders/1715177193492712?o=8362622574829379#)

---

***Estudo***: Cluster De Migração

<details>
<summary>Qual cluster devo utilizar?</summary>
A máquina mais apropria que eu aconselho a ser utilizada é a maquina da família ‘Série E`, no caso a 'Standard_E8as_v4’'. Pois a migração vai ser a junção de diversos gigas de dados.
A família de máquinas virtuais "E"  é projetada para cargas de trabalho intensivas em CPU e é conhecida por oferecer um equilíbrio sólido entre CPU, memória e armazenamento. Ela pode ser adequada em várias situações;
8 vCPUs: Isso pode ser adequado para processar operações em paralelo e acelerar as tarefas de migração, especialmente se o processo de migração for paralelizável.
64 GB de memória: Uma quantidade razoável de memória, que pode ser benéfica para processamento eficiente, principalmente se as operações envolverem transformações ou agregações complexas nos dados.
Armazenamento SSD: As instâncias da série "E" geralmente usam discos SSD, o que é vantajoso para operações de leitura e gravação mais rápidas, que são frequentemente envolvidas em migrações de dados.

Acredito que no primeiro momento podemos manter a família “E”, mas se começarmos enfrentar dificuldades ou lentidão, podemos tentar utilizar a família de máquinas virtuais "Série D" (Geral Uso Balanceado). Essa família oferece um equilíbrio entre CPU, memória e armazenamento, o que pode ser também adequado para tarefas de migração de dados.

A questão do ambiente, aconselho utilizar dbw-nonprod-iris-intelligence/dbw-prod-iris-intelligence, para trabalhar na questão de migração, criando um cluster próprio para a tarefa de migração, único ponto que fica, é que devemos ter os Mounts criados para fazer a busca no dado informado pelo usuário, acredito que em nonprod-iris-intelligence, não temos o Mount criado para a rawzone, que seria necessário também, efetuar a migração tanto de dev, quanto de Prod!

**Obs:** Ainda não foi criado nenhum cluster ou mounts para migração, acredito que isso ficara para ser criado no primeiro processo de migração, junto com os mounts da brewzone, quando tiverem prontos. Para não ter retrabalho.

Especificações Standard_E8as_v4: https://azureprice.net/vm/Standard_E8as_v4 

</details>

---

***Vídeo Tutorial***: Para compreender plenamente o tópico e aprender como realizar uma migração de dados completa, preparei um vídeo que detalha todos os aspectos do processo. É essencial que você reserve um tempo para assistir atentamente a este vídeo e, além disso, recomendo a leitura da documentação sobre o procedimento de migração, onde também encontrará um tutorial mais aprofundado sobre o processo de migração.

<iframe src="https://anheuserbuschinbev.sharepoint.com/sites/irisplatform/_layouts/15/embed.aspx?UniqueId=47d20699-7f66-49cb-985e-9754559d6dc1&embed=%7B%22ust%22%3Atrue%2C%22hv%22%3A%22CopyEmbedCode%22%7D&referrer=StreamWebApp&referrerScenario=EmbedDialog.Create" width="640" height="360" frameborder="0" scrolling="no" allowfullscreen title="Migracao Card Operacao.mp4"></iframe>


> **Importante**: Lembre-se sempre de realizar a migração em ambas as camadas, tanto na **'rawzone'** quanto na **'trustedzone'**. Além disso, nunca exclua nenhum dado sem a aprovação da equipe que solicitou a migração dos dados. Estamos à disposição para esclarecer qualquer dúvida e ajudar a solucionar eventuais problemas que possam surgir.
{.is-warning}