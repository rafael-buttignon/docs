# Caminhos Dinâmicos

Os conectores de SMB e ABFS contam com o recurso de caminhos dinâmicos, permitindo que a ingestão do dado seja parametrizada de acordo com os valores informados no YAML da DAG. Essa transformação acontece em tempo de execução, e permite customizar o diretório do arquivo tomando como base datas.

## Como este recurso funciona

O usuário poderá informar os seguintes campos no conector de SMB ou ABFS:

~~~yaml
file_path: "Brazil/source/:date1:/:date2:/arquivo.csv"
path_variables:
  date1: 
  	day_range: 0
    date_format: "%Y%m"
  date2:
    day_range: 1
    date_format: "%Y%m%d"
~~~

Ao informar a variável tendo **date** como início de nomenclatura, será entendido que este parâmetro será customizado de acordo com as regras implementadas para lidar com data. 

- <kbd>file_path</kbd>: é o caminho/diretório aonde o seu arquivo está contido. Para utilizar o recurso de caminho dinâmico, é preciso que tenha ao menos uma variável a ser customizada. Elas são representadas entre dois-pontos "**:**". 
Exemplo: <kbd>"Brazil/source/:date1:/:date2:/arquivo.csv"</kbd>
 	- <kbd>**Validação**</kbd>: obrigatório, não nulo, string.
---
- <kbd>path_variables</kbd>: dicionário contendo todas as variáveis a serem customizadas pelo caminho dinâmico.

  - <kbd>Validação</kbd>: obrigatório, não nulo e dicionário.
---
- <kbd>day_range</kbd>: caso a variável no **path_variables** comece com _date_, este parâmetro entenderá o dia atual subtraindo n dias, informada aqui. 
Exemplo: se o dia atual for 27 de Março de 2023 (2023-03-27) e o parâmetro estiver configurado como 1, o caminho será entendido como 2023-03-26

  - <kbd>Validação</kbd>: obrigatório se passado uma variávei _date_, não nulo e int.
---
- <kbd>date_format</kbd>: caso a variável no **path_variables** comece com _date_, este parâmetro  irá alterar o formato em que a data será disposta. 
Exemplo: se passada uma string no formato *%Y-%d-%m* e a data atual for 2023-03-27, o caminho final será 2023-27-03

  - <kbd>Validacão</kbd>: obrigatório se passado uma variávei _date_, não nulo e string. Por padrão de sistema, o valor é *ANO-MES-DIA (%Y%m%d)*.


Neste exemplo, sendo o dia atual 17 de Abril de 2023, o caminho retornado será: <kbd>'Brazil/source/202304/20230416/arquivo.csv'</kbd>

>**Observação:** Caso não seja informado o nome do arquivo no *file_path*, será retornado o **último** arquivo do diretório. Desta forma, se o seu caminho estiver no formato <kbd>'Brazil/source/202304/20230416/'</kbd> , o connector irá varrer este diretório e retornar o último arquivo contido nele. 
{.is-info}
