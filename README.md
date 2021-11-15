### Conectores Spring Cloud e Heroku

# 1. Visão Geral
Neste artigo, vamos cobrir a configuração de um aplicativo Spring Boot no Heroku usando Spring Cloud Connectors.

Heroku é um serviço de hospedagem de serviços web. Além disso, eles fornecem uma grande seleção de serviços de terceiros, chamados de complementos, que fornecem tudo, desde o monitoramento do sistema até o armazenamento do banco de dados.

Além de tudo isso, eles têm um pipeline de CI / CD personalizado que se integra perfeitamente ao Git que agiliza o desenvolvimento para a produção.

O Spring oferece suporte ao Heroku por meio de sua biblioteca Spring Cloud Connectors. Usaremos isso para configurar uma fonte de dados PostgreSQL em nosso aplicativo automaticamente.

Vamos começar a escrever o aplicativo.

# 2. Serviço Spring Boot Book
Primeiro, vamos criar um novo serviço Spring Boot simples.

# 3. Inscrição no Heroku
Agora, precisamos nos inscrever para uma conta Heroku. Vamos para heroku.com e clique no botão de inscrição no canto superior direito da página.

Agora que temos uma conta, precisamos obter a ferramenta CLI. Precisamos navegar até a página de instalação do heroku-cli e instalar este software. Isso nos dará as ferramentas de que precisamos para concluir este tutorial.

# 4. Crie o aplicativo Heroku
Agora que temos a CLI do Heroku, vamos voltar ao nosso aplicativo.

### 4.1. Inicializar Repositório Git
Heroku funciona melhor ao usar git como nosso controle de origem.

Vamos começar indo para a raiz de nosso aplicativo, o mesmo diretório de nosso arquivo pom.xml, e executando o comando git init para criar um repositório git. Em seguida, execute git add. e git commit -m “primeiro commit”.

Agora temos nosso aplicativo salvo em nosso repositório git local.

### 4.2. Provisionar Heroku Web App
A seguir, vamos usar a CLI do Heroku para provisionar um servidor web em nossa conta.

Primeiro, precisamos autenticar nossa conta Heroku. Na linha de comando, execute heroku login e siga as instruções para efetuar login e criar uma chave SSH.

Em seguida, execute heroku create. Isso irá provisionar o servidor da web e adicionar um repositório remoto ao qual podemos enviar nosso código para implantações. Também veremos um domínio impresso no console, copie este domínio para que possamos acessá-lo mais tarde.

### 4.3. Envie o código para o Heroku
Agora usaremos o git para enviar nosso código para o novo repositório Heroku.

Execute o comando git push heroku master para enviar nosso código ao Heroku.

Na saída do console, devemos ver logs indicando que o upload foi bem-sucedido, então o sistema irá baixar quaisquer dependências, construir nosso aplicativo, executar testes (se houver) e implantar o aplicativo se tudo correr bem.

É isso - agora temos nosso aplicativo implementado publicamente em um servidor web.

# 5. Teste In-Memory no Heroku
Vamos verificar se nosso aplicativo está funcionando. Usando o domínio de nossa etapa de criação, vamos testar nosso aplicativo ativo.

Vamos emitir algumas solicitações HTTP:

```
POST https://{heroku-domain}/books HTTP
{"author":"isaccanedo","title":"Spring Boot on Heroku"}
```

Devemos voltar:

```
{
    "title": "Spring Boot on Heroku",
    "author": "isaccanedo"
}
```

Agora vamos tentar ler o objeto que acabamos de criar:

```
GET https://{heroku-domain}/books/1 HTTP
```

Isso deve retornar:

```
{
    "id": 1,
    "title": "Spring Boot on Heroku",
    "author": "isaccanedo"
}
```

Tudo parece bom, mas na produção, devemos usar um armazenamento de dados permanente.

Vamos percorrer o provisionamento de um banco de dados PostgreSQL e a configuração de nosso aplicativo Spring para usá-lo.

# 6. Adicionando PostgreSQL
Para adicionar o banco de dados PostgreSQL, execute este comando heroku addons: create heroku-postgresql: hobby-dev

Isso fornecerá um banco de dados para nosso servidor da web e adicionará uma variável de ambiente que fornece as informações de conexão.

O Spring Cloud Connector é configurado para detectar essa variável e configurar a fonte de dados automaticamente, já que o Spring pode detectar que queremos usar PostgreSQL.

Para informar ao Spring Boot que estamos usando PostgreSQL, precisamos fazer duas alterações.

Primeiro, precisamos adicionar uma dependência para incluir os drivers PostgreSQL:

```
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.2.10</version>
</dependency>
```

A seguir, vamos adicionar propriedades para que o Spring Data Connectors possa configurar o banco de dados de acordo com seus recursos disponíveis.

Em src/main/resources, crie um arquivo application.properties e adicione as seguintes propriedades:

```
spring.datasource.driverClassName=org.postgresql.Driver
spring.datasource.maxActive=10
spring.datasource.maxIdle=5
spring.datasource.minIdle=2
spring.datasource.initialSize=5
spring.datasource.removeAbandoned=true
spring.jpa.hibernate.ddl-auto=create
```

Isso agrupará nossas conexões de banco de dados e limitará as conexões de nosso aplicativo. O Heroku limita o número de conexões ativas em um banco de dados da camada de desenvolvimento a 10 e, portanto, definimos nosso máximo como 10. Além disso, definimos a propriedade hibernate.ddl para criar de modo que nossa tabela de livros seja criada automaticamente.

Finalmente, faça commit dessas mudanças e execute git push heroku master. Isso enviará essas alterações para nosso aplicativo Heroku. Depois que nosso aplicativo for iniciado, tente executar os testes da seção anterior.

A última coisa que precisamos fazer é alterar a configuração ddl. Vamos atualizar esse valor também:

```
spring.jpa.hibernate.ddl-auto=update
```

Isso instruirá o aplicativo a atualizar o esquema quando forem feitas alterações na entidade quando o aplicativo for reiniciado. Comprometa e impulsiona esta mudança como antes para que as mudanças sejam enviadas para nosso aplicativo Heroku.

Não precisamos escrever uma integração de fonte de dados customizada para nada disso. Isso porque o Spring Cloud Connectors detecta que estamos executando com Heroku e usando PostgreSQL - e conecta automaticamente a fonte de dados Heroku.

# 5. Conclusão
Agora temos um aplicativo Spring Boot em execução no Heroku.

Acima de tudo, a simplicidade de ir de uma única ideia a um aplicativo em execução torna o Heroku uma maneira sólida de implantar.