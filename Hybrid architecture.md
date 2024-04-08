# Arquitetura híbrida

Dada a necessidade de viabilisarmos os cenários híbridos na arquitetura 2.0, isto é, cenários em que parte do fluxo, desde a ingestão até o serving, está na 1.0 e outra parte está na 2.0. dotamos o fluxo abaixo quanto a tomada de decisão da criação, ou não, de novos produtos de dados na arquitetura 2.0 utilizando tais cenários. 

## Fluxo de decisão

![Iris - Cenários híbridos](https://i.imgur.com/eR2TAqU.jpg =1000x)

### Ingestões na 1.0 e transformações na 2.0
Caso o time da IRIS esteja em falta com um dos conectores necessários para a ingestão, deverá ser alinhado com o time de IRIS Ingestion se há a possibilidade de aguardar o desenvolvimento do conector para então dar início ao desenvolvimento. Em caso negativo quanto a possibilidade da espera, será proposta a viabilidade do time da torre de Analytics contribuir no desenvolvimento do conector, em caso de uma nova negativa, será então permitida a ingestão na 1.0 e a transformação na 2.0.
Além disso, caso o dado já esteja na HZ ou na CZ e não haja a possibilidade da migração da ingestão para a 2.0, seja por falta de conector ou por conflito com outras demandas, também será permitido consumir os dados da HZ e da CZ por meio da arquitetura 2.0 de transformation. Contudo, para este último cenário se faz necessário alinhar com o time de IRIS Transformation quando a migração dos dados da 1.0 seria possível.

### Ingestões na 2.0 e transformações na 1.0
Caso o time da IRIS esteja em falta com alguma das ferramentas de serving, será permitido realizar as transformação nos workspaces da 1.0 enquanto a ingestão é feita na 2.0, após a limitação ser alinhada com a Iris.

> **Atenção:** Para qualquer um dos cenários é necessário informar no Pull Request qual a **data de previsão da migração completa**, além de apresentar uma **evidência** em forma de print de que o gerente da área está *"OK"* com o desenvolvimento de um dataset híbrido.
{.is-warning}


## Dúvidas? 
Em caso de dúvidas entre em contato conosco através do nosso  <a href="https://dev.azure.com/AMBEV-SA/IRIS-Analytics%20Platform/_boards/board/t/AnalyticsTeam/Customer%20Service" target="_blank" >**Board de operações**</a>.
