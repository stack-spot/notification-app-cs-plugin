#### **Inputs**

* Region - Região da AWS em que o serviço SNS foi criado - Campo Obrigatório.

Você pode sobrescrever a configuração padrão adicionando a seção `Sns` em seu `appsettings.json`.

```json
  "Sns": {
      "Region": "us-east-1"
  }
```

> É possivel adicionar nessa seção o parâmetro `topicArn` para comunicação com o seu tópico. - Não Obrigatório.
```json
  "Sns": {
      "Region": "us-east-1",
      "TopicArn": "arn:aws:sns:us-east-1:000000000000:mytopic",
  }
```

#### 3. Adicione ao seu `IServiceCollection` via `services.AddNotificationSns()` no `Startup` da aplicação ou `Program` tendo como parametro de entrada `IConfiguration` e `IWebHostEnvironment`. 

```csharp
//using StackSpot.Notification.SNS;

services.AddNotificationSns(configuration);
```
> Você tem disponível as opções para customizar o seu serviço utilizando os parâmetros e sobrecargas do método `AddNotificationSns` para `ServiceLifetime` e `AWSOptions`.

##### Implementação

* A  classe da mensagem que será enviada para o serviço de notificação, deverá herdar da classe `SnsMessage`.
* Publish - Publica uma mensagem no serviço de notificação. Por padrão utiliza o Arn do tópico configurado no `appsetings.json`. Ele irá retornar o `MessageId` em caso de sucesso.

```csharp
// using StackSpot.Notification.SNS.Common;
// using StackSpot.Notification.SNS;

public class Message : SnsMessage
{
    public Message(string text)
    {
        _text = text;
    }

    private readonly string _text;

    public string text
    {
        get { return _text; }
    }
}

[ApiController]
[Route("[controller]")]
public class SampleController : ControllerBase
{
    private readonly INotification<SnsMessage> _sns;

    public SampleController(INotification<SnsMessage> sns)
    {
        _sns = sns;
    }

    [HttpPost]
    public async Task<IActionResult> Post([FromBody] string text)
    {        
        var message = new Message(text);
        var result = await _sns.Publish(message);
        
        return Ok($"MessageId: {result.Content}");
    }
}
```
> Adicionalmente ao plubicar uma mensagem você pode utilizar a sobrecarga do método `Publish` e informar para qual `Tópico` será enviada a mensagem através do parâmetro `topicArn`.

* A interface `INotification` também disponibiliza os métodos `ListTopics` e `ListSubscriptionsByTopic` que listam os Tópicos e Inscritos respectivamente.

#### 4. Ambiente local

* Esta etapa não é obrigatória.
* Recomendamos, para o desenvolvimento local, a criação de um contâiner com a imagem do [Localstack](https://github.com/localstack/localstack). 
* Para o funcionamento local você deve preencher a variável de ambiente `LOCALSTACK_CUSTOM_SERVICE_URL` com o valor da url do serviço. O valor padrão do localstack é http://localhost:4566.
* Abaixo um exemplo de arquivo `docker-compose` com a criação do contâiner: 

```yaml
version: '2.1'

services:
  localstack:
    image: localstack/localstack
    ports:
      - "4566:4566"
    environment:
      - SERVICES=sns
      - AWS_DEFAULT_OUTPUT=json
      - DEFAULT_REGION=us-east-1
```

Após a criação do contâiner, crie um tópico para realizar os testes com o componente. Recomendamos que você tenha instalado em sua estação o [AWS CLI](https://aws.amazon.com/pt/cli/). Abaixo um exemplo de comando para criação de uma fila:

```bash
aws sns create-topic --endpoint-url=http://localhost:4566 --region=us-east-1 --name [NOME DO SEU TÓPCIO]
