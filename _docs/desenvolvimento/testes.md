---
title: Testes
category: Operações
order: 3
type: 4
---

Os componentes de software que proporcionam endpoints HTTP REST, incluídos no nosso pacote de software, têm desenvolvidos testes de integração Quarkus.

## Esquema de Estratégia de Testes

![Alt text](/images/posts/es_tests.png?raw=true "Title")


### Testes de Integração

O padrão geral seguido nos testes de integração desenvolvidos para os componentes Consumer (Quarkus) da arquitetura foi a injeção de mensagens com conteúdo fixo e conhecido nos inputs dos serviços e a verificação da correção dos respetivos outputs.

Como tal, as mensagens são injetadas nos In-Memory Channels, que substituem os tópicos Kafka em contexto de testes, e são efetuadas as seguintes validações:

- Chamada ao endpoint SSE /stream de forma a confirmar se o pedido é aceite, se o output existe, e se corresponde ao valor expectável.
- Chamada ao endpoint REST /all de modo a, de forma análoga, verificar se o pedido é aceite, se o output existe, e se corresponde ao valor expectável. De notar que este teste engloba todo o fluxo de leitura e escrita na base de dados InfluxDB, pelo que todas essas operações são validadas caso o teste conclua com sucesso.
- Leitura, nos casos onde aplicável, das mensagens enviadas pelo módulo para os In-Memory Channels que substituem tópicos Kafka como 'events' e 'arrived'. Esses envios são igualmente comparados com o resultado expectável por forma a confirmar o bom-funcionamento do módulo e da lógica inerente. 

### Testes Unitários

Foram igualmente desenvolvidos testes unitários (JUnit) para validar lógica de processamento em dois dos componentes integrais da arquitetura.
Estes são executados pelo plugin surefire do Maven a cada operação de build ou test, incluíndo na pipeline Jenkins.

#### EnvironmentConsumer

A função EnvironmentKafkaConsumer.parse recebe os dados consumidos e aplica transformações ao JSON recebido de modo a omitir informação irrelevante e alterar o formato dos dados.
Para validar o seu funcionamento foi desenvolvido o teste unitário 'EnvironmentKafkaConsumerTest.parse', que executa o método com um argumento cujo resultado é conhecido, verificando posteriormente a correção do valor de retorno.

#### ArrivedConsumer

A função MainResource.process recebe os dados consumidos e aplica lógica de processamento ao JSON recebido de modo a obter listas com os novos tipos de eventos a lançar.
Para validar o seu funcionamento foi desenvolvido o teste unitário 'MainResourceTest.process', que executa o método com um argumento cujos resultados são conhecido, verificando posteriormente a correção do conteúdo das várias listas resultantes de eventos e mensagens a lançar.

## Dependências de dados externas

Os componentes desenvolvidos a testar, que disponibilizam as chamadas para os serviços, contêm dependências de dados: de modo a testar o módulo de maneira independente de recursos exteriores, para garantir que apenas a aplicação em questão está a ser testada, é necessário que se desenvolva algo que substitua estas chamadas à base de dados. 

### Kafka e InMemoryConnector

No que toca à dependência exterior Kafka, foram usadas as funcionalidade do Quarkus para a criação de canais em memória (denominados, normalmente, como *In-Memory Channels*), que substituam os tópicos a aceder no *broker* Kafka externo. Assim, cada aplicação a ser testada tem uma classe de recurso Kafka denominada **TestKafkaResource**, que implementa a interface **QuarkusTestResourceLifecycleManager** – esta interface disponibilizada pelo ambiente Quarkus garante que estes recursos de teste sejam **começados aquando da execução de cada classe de teste** e, no final, **eliminados e limpos** do ambiente de execução, mesmo antes do final de cada classe. Isto é obtido através da implementação dos métodos **start()** e **stop()**. O primeiro método retorna sempre um mapa de configurações de sistema, de modo a configurar essas propriedades ao ambiente aplicacional após a sua inicialização.

```
public class TestKafkaResource implements QuarkusTestResourceLifecycleManager {
    @Override
    public Map<String, String> start() {
        ...
```

O método *start()*, neste caso, é implementado para passar **todos os canais** que estão anotados com **@Incoming** e **@Outgoing** (mensagens que provêm da subscrição de tópicos no broker ou consumidos destes) como **canais *In-Memory***. Isto é efetuado a partir das funções estática do InMemoryConnector **switchIncomingChannelsToInMemory()/ switchOutgoingChannelsToInMemory()**. No final, são retornadas as propriedades aplicacionais de teste adicionais, para serem reescritas e utilizadas posteriormente no ciclo de vida do teste.

```
...
Map<String, String> props1 = InMemoryConnector.switchIncomingChannelsToInMemory("flights");
Map<String, String> props2 = InMemoryConnector.switchOutgoingChannelsToInMemory("sse-flights");
...
```

O método *stop()* faz a limpeza dos canais em memória com o método **InMemoryConnector.clear()**.

### InfluxDB e Testcontainers

Em relação ao recurso InfluxDB, esta dependência é mais rebuscada, pois necessita de um endpoint e de uma ligação específica ao serviço aquando do teste, para serem utilizados nos pedidos ou inserções à base de dados. Assim sendo, o método utilizado para criar um recurso de testes deste tipo foi com recurso a um **Testcontainer** genérico, que inicia no Docker *daemon* da máquina em execução, e inicializa um **container volátil durante o ciclo de vida da classe de teste** (implementa também a interface QuarkusTestResourceLifecycleManager).

Neste recurso de teste, **TestInfluxDBResource**, a sua inicialização é feita configurando variáveis de ambiente ao ambiente do container Docker, similares às inseridas no ambiente em produção. Depois, o **container é criado** usando essas variáveis de ambiente, **expondo o porto de teste** 8086. Por fim, é retornada a **propriedade de execução com o URL de InfluxDB** correspondente (apontando assim o módulo de testes para o container criado), para ser utilizado pela classe. Este é construído a partir do **endereço de IP do container** e o **porto exposto** mapeado. 

```
...
vars.put("influx.url", "http://" + influxDBContainer.getContainerIpAddress() + ":" + influxDBContainer.getFirstMappedPort());
return vars;
```

No final do ciclo de vida do teste (*stop()*), **o container volátil é parado**.
No teste, agora, o **InMemoryConnector** deve ser **injetado**, e utilizado, na inicialização deste (com a anotação *@BeforeAll*), para inserir os dados antes da execução dos testes – **isto é uma pré-condição do teste de integração**, e deve ser feita de modo a dar o conjunto de dados de teste necessários à sua execução.

O teste aos endpoints de *stream*, **do tipo SSE** (*Server-sent events*) utilizam uma função desenvolvida para o efeito (**connectToSSE()**) que mapeia uma zona de memória que possa ser acedida pelo programa de testes a um registo de consumo assíncrono da stream de dados. Assim, é possível criar uma thread que se **regista no canal de *stream*** pedido e **vai inserindo as mensagens** à zona de memória. Neste caso, é uma **lista** declarada como campo da nossa classe de testes, que é inicializada no início da função, e que, **por cada evento de dados proveniente na *stream*, insere-o nessa lista**.

```
this.sseMessages = new ArrayList<>();
Client client = ClientBuilder.newClient();
WebTarget target = client.target("http://localhost:19001/flights/stream");

source = SseEventSource.target(target).build();
source.register(e -> sseMessages.add(e.readData()));
source.open();
...
```

### Anotações nas classes de testes

A classe de teste é **anotada como um teste Quarkus**, e são também **definidos os recursos de teste** a ser usados. 

```
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@QuarkusTestResource(value = TestKafkaResource.class, parallel = true)
@QuarkusTestResource(value = TestInfluxDBResource.class, parallel = true)
@QuarkusTest
public class FlightsKafkaConsumerTest {
...
```

Cada teste de integração (ou seja, cada método correspondente a cada endpoint HTTP), é executado na classe de testes de cada aplicação Quarkus. Assim, todos os testes da API REST deste sub-módulo pertencem a este ciclo de vida de testes.

### Configuração do servidor HTTP

Em termos da configuração de testes do servidor HTTP, no application.properties referente ao âmbito de testes foi configurado para utilizar um **porto de teste**, em vez do porto configurado em produção ou o porto por defeito normalmente utilizado. Também são configurados **timeouts às chamadas aos pontos HTTP a testar** para não ficarem em *deadlock* infinito (útil para situações em que o teste nunca acaba, como o caso endpoints de *stream*, e então a conexão é abortada por *timeout*), e ainda um **timeout aquando da deteção da paragem** da execução do teste, por outras razões que possam fazer parar a execução do teste.

```
quarkus.http.test-port=19001
quarkus.http.test-timeout=60s
quarkus.test.hang-detection-timeout=30s
```
