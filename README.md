#  Tópicos do Projeto

1 Telegram

2 Ingestão

3 ETL

4 Apresentação

5 Storytelling


# 1 Telegram

**Telegram** é uma plataforma de mensagens instantâneas *freeware* (distribuído gratuitamente) e, em sua maioria, *open source*. É muito popular entre desenvolvedores por ser pioneiro na implantação da funcionalidade de criação de **chatbots**, que, por sua vez, permitem a criação de diversas automações. 


1.1. Primeiros passos

> 1 - Se você ainda não tiver uma conta no Telegram, crie uma e faça o login na versão web da plataforma acessando este [link](https://web.telegram.org).

> 2 - Crie um bot.

> 3 - Crie um grupo e adicione o bot

> 4 - Transforme o bot em um administrador do grupo.

> 5 - Desabilite a opção de adicionar o bot a novos grupos.

> 6 - Envie mensagens no grupo e consuma utilizando a API de bots do **Telegram**.


# A biblioteca getpass permite que você utilize senhas ou dados sensíveis de forma segura

from getpass import getpass

token = getpass()

## 2 Ingestão

A etapa de **ingestão** tem como principal função, como o nome sugere, a captura dos dados transacionais para ambientes analíticos. Em geral, os dados ingeridos são armazenados no formato mais próximo do original, sem qualquer transformação em seu conteúdo ou estrutura (schema). Por exemplo, dados provenientes de uma API web que adota o padrão REST (representational state transfer) são recebidos e, em seguida, armazenados no formato JSON.

> 2.1 Crie um `bucket` no `AWS S3` para o armazenamento de dados crus, não se esqueça de adicionar o sufixo `-raw`.

> 2.2. Crie uma função no `AWS Lambda` para recebimento das mensagens e armazenamento no formato JSON no `bucket` de dados crus. Não se esqueça de configurar as variáveis de ambiente e de adicionar as permissão de interação com `AWS S3` no `AWS IAM`.

**Nota**: Para testar a função com evento do próprio `AWS Lambda`, substitua o código `message = json.loads(event["body"])` por `message = event`. Lembre-se que o primeiro só faz sentido na integração com o `AWS API Gateway`. O códigoestá disponível em aws_lambda.ipynb

> 2.3. Crie uma API no `AWS API Gateway` a conecte a função do `AWS Lambda`.

**Nota**: não disponibilize o endereço da API gerada.

aws_api_gateway_url = getpass()

> 2.4. Configura o *webhook* do *bot* através do método `setWebhook` da API de *bots* do **Telegram**. utilize o endereço da API criada no `AWS API Gateway`. Utilize o método `getWebhookInfo` para consultar a integração.

**Nota**: não disponibilize o *token* de acesso ao seu *bot* da API de *bots* do **Telegram**.

## 3 ETL 

A etapa de **extração, transformação e carregamento (ETL, do inglês *extraction, transformation, and load*)** é um processo abrangente responsável por manipular os dados que foram ingeridos de sistemas transacionais, ou seja, dados já armazenados em camadas brutas ou raw em sistemas analíticos. Os processos realizados nessa fase podem variar significativamente conforme a área da empresa, o volume, a variedade e a velocidade dos dados consumidos, entre outros fatores. De forma geral, o dado bruto passa por um processo contínuo de data wrangling, no qual ele é limpo, deduplicado e ajustado, sendo então armazenado utilizando técnicas como particionamento, modelagem em colunas e compressão. Após esse processamento, os dados ficam prontos para análise por profissionais especializados.

> 3.1. Crie um `bucket` no `AWS S3` para o armazenamento de dados enriquecidos, não se esqueça de adicionar o sufixo `-enriched`.

> 3.2. Crie uma função no `AWS Lambda` para processar as mensagens JSON de uma única partição do dia anterior (D-1), armazenadas no *bucket* de dados crus. Salve o resultado em um único arquivo PARQUET, também particionado por dia. Para isso, você vai utilizar o aws_labda_enriched.py. Não se esqueça de configurar as variáveis de ambiente, de adicionar as permissão de interação com `AWS S3` no `AWS IAM`, de configurar o *timeout* e de adicionar a *layer* com o código do pacote Python PyArrow.

**Nota**: Para testar a função, substitua o código `date = (datetime.now(tzinfo) - timedelta(days=1)).strftime('%Y-%m-%d')` por `date = (datetime.now(tzinfo) - timedelta(days=0)).strftime('%Y-%m-%d')`, permitindo assim o processamento de mensagens de um mesmo dia.

> 3.3. Crie uma regra no `AWS Event Bridge` para executar a função do `AWS Lambda` todo dia a meia noite no horário de Brasília (GMT-3).

## 4 Apresentação

Na etapa de **apresentação**, os dados serão exibidos por meio da interface SQL para análise. Essa interface é apresentada em uma tabela externa, acessando os dados que estão armazenados na camada mais refinada da arquitetura, conhecida como camada enriquecida.

> 4.1 AWS Athena

```sql
CREATE EXTERNAL TABLE `telegram`(
  `message_id` bigint, 
  `user_id` bigint, 
  `user_is_bot` boolean, 
  `user_first_name` string, 
  `chat_id` bigint, 
  `chat_type` string, 
  `text` string, 
  `date` bigint)
PARTITIONED BY ( 
  `context_date` date)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat'
LOCATION
  's3://<bucket-enriquecido>/'
```

Por fim, adicione as partições disponíveis.

> **Importante**: Toda vez que uma nova partição é adicionada ao repositório de dados, é necessário informar o `AWS Athena` para que a ela esteja disponível via SQL. Para isso, use o comando SQL `MSCK REPAIR TABLE <nome-tabela>` para todas as partições (mais caro) ou `ALTER TABLE <nome-tabela> ADD PARTITION <coluna-partição> = <valor-partição>` para uma única partição (mais barato), documentação neste [link](https://docs.aws.amazon.com/athena/latest/ug/alter-table-add-partition.html)).

```sql
MSCK REPAIR TABLE `telegram`;
```
E consulte as 10 primeiras linhas para observar o resultado.

```sql
SELECT * FROM `telegram` LIMIT 10;
```

> 4.3 Analytics

- Quantidade de mensagens por dia.

```sql
SELECT 
  context_date, 
  count(1) AS "message_amount" 
FROM "telegram" 
GROUP BY context_date 
ORDER BY context_date DESC
```

- Quantidade de mensagens por usuário por dia.

```sql
SELECT 
  user_id, 
  user_first_name, 
  context_date, 
  count(1) AS "message_amount" 
FROM "telegram" 
GROUP BY 
  user_id, 
  user_first_name, 
  context_date 
ORDER BY context_date DESC
```

- Média do tamanho das mensagens por usuário por dia.

```sql
SELECT 
  user_id, 
  user_first_name, 
  context_date,
  CAST(AVG(length(text)) AS INT) AS "average_message_length" 
FROM "telegram" 
GROUP BY 
  user_id, 
  user_first_name, 
  context_date 
ORDER BY context_date DESC
```

- Quantidade de mensagens por hora por dia da semana por número da semana.

```sql
WITH 
parsed_date_cte AS (
    SELECT 
        *, 
        CAST(date_format(from_unixtime("date"),'%Y-%m-%d %H:%i:%s') AS timestamp) AS parsed_date
    FROM "telegram" 
),
hour_week_cte AS (
    SELECT
        *,
        EXTRACT(hour FROM parsed_date) AS parsed_date_hour,
        EXTRACT(dow FROM parsed_date) AS parsed_date_weekday,
        EXTRACT(week FROM parsed_date) AS parsed_date_weeknum
    FROM parsed_date_cte
)
SELECT
    parsed_date_hour,
    parsed_date_weekday,
    parsed_date_weeknum,
    count(1) AS "message_amount" 
FROM hour_week_cte
GROUP BY
    parsed_date_hour,
    parsed_date_weekday,
    parsed_date_weeknum
ORDER BY
    parsed_date_weeknum,
    parsed_date_weekday
```
