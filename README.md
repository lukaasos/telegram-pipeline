#  Tópicos do Projeto

1 Telegram

2 Ingestão

3 ETL

4 Apresentação

5 Storytelling


# Telegram

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

> 2.1 Crie um `bucket` no `AWS S3` para o armazenamento de dados crus, não se esqueça de adicionar o sufixo `-raw`.

> 2.2. Crie uma função no `AWS Lambda` para recebimento das mensagens e armazenamento no formato JSON no `bucket` de dados crus. Não se esqueça de configurar as variáveis de ambiente e de adicionar as permissão de interação com `AWS S3` no `AWS IAM`.

**Nota**: Para testar a função com evento do próprio `AWS Lambda`, substitua o código `message = json.loads(event["body"])` por `message = event`. Lembre-se que o primeiro só faz sentido na integração com o `AWS API Gateway`. O códigoestá disponível em aws_lambda.ipynb

