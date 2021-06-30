---
title: Logging agregado
category: Operações
order: 5
type: 4
published: true
---

### Overview

A monitorização centralizada aos registos de logging gerados pelas aplicações é feita a partir da instalação e configuração de uma **stack EFK (Elasticsearch, Fluentd, Kibana)**. Embora o Logstash seja o agregador de registos de logging mais utilizado, este modelo tem a particularidade de utilizar o **Fluentd** na sua vez, sendo que a sua configuração de cada fonte de dados é **mais direta**, e o **formato de configuração é mais visualizável** dentro do ficheiro.

A monitorização de logging é necessária para se observar simultaneamente um número elevado de aplicações Java, que estão constantemente a produzir logs a um rácio significativo, por exemplo, significando que poderá existir ali um problema com o micro-serviço, como um número muito elevado de exceções *runtime* no código Java, o que pode revelar um problema com o código ou de comunicação entre os vários serviços que este depende. Mais ainda, a visualização de logs a partir de um ponto central **permite maior reação à ocorrência de problema**, sendo que os logs de vários serviços podem ser **visualizados ao mesmo tempo**, ou com filtros a partir de botões na interface da dashboard. O armazenamento destes registos permite a **recolha de informação histórica** do funcionamento da aplicação, necessária para, por exemplo, **auditorias**.

### Stack

A stack é fornecida dentro do pacote de software do projeto, a partir da instalação *docker-compose*. Estes componentes ligam-se naturalmente entre si, não precisando de grandes configurações, sendo que estão dentro de containers e as configurações mínimas já estão garantidas.

![efk-stack-flux-model](/images/spec/es_log_architecture.png?raw=true "EFK flux model")

O modelo de agregação de logs ilustra que os vários componentes de software Java enviam os seus logs para o agregador *Fluentd* no **formato Syslog**, que expõe um porto e recebe os vários registos sob a forma de mensagens, enviando depois estes registos recebidos para o serviço **Elasticsearch**. Esta abordagem é muito simples de implementar, no entanto **pode sofrer de bottleneck em termos de performance e prontidão do envio das mensagens** para o Elasticsearch em “tempo real”, pois com a escalabilidade do número de serviços existentes a monitorizar, poderá fazer com que um container de Fluentd não seja suficiente para o efeito; assim como **poderá resultar em perda de registos** de logging, **devido ao protocolo UDP** que é utilizado na transmissão de logs entre as aplicações e o agregador.

### Integração ao software

As aplicações Quarkus são configuradas para enviar os seus logs a partir da configuração de parâmetros nos ficheiros *“application.properties”* correspondentes. **A integração com a *stack* EFK com Quarkus é, então, trivial**, superiorizando-se a aplicações com base em Spring Boot, que requerem a criação de *appenders*.

![log-config-quarkus-example](/images/spec/log_config_app_quarkus.PNG?raw=true "Example of the Quarkus logging integration with the log aggregator")

### Visualização de logs

A ferramenta de visualização escolhida é o **Kibana**, pois está **naturalmente integrada** como parte da stack de logs centralizados, sendo que a visualização de logs e suas estatísticas base são facilmente criadas em **painéis**, e consequentemente **dashboards**.

A dashboard de visualização realizada serve para ter o mínimo de informação prontamente disponível que se achou útil: em primeiro, uma **tabela** que mostra **todos os registos de todos os logs** da infraestrtura mediante o **intervalo de tempo** filtrado, que inclui, para além da *mensagem* os campos do *nome dos serviços* (ou seja, das aplicações) e o *timestamp* da sua ocorrência; depois, existe ainda um **gráfico de barras da contagem de logs totais**, nas últimas doze horas. Por fim, existe um painel de **visualização de proporção de logs de cada aplicação** a partir do **tamanho da letra** (quanto maior a letra em relação aos outros nomes, mais logs está a produzir comparativamente).

Estes painéis, combinado com os filtros que estão integrados nas páginas das dashboards, permite rapidamente **flitrar por aplicação** ou aplicações, **limitar intervalos** de tempo para visualização, entre outros. Assim, a monitorização de logging atinge o nível de controlo desejado.

![kibana-dashboard-ss](/images/spec/kibana_dashboard.PNG?raw=true "Kibana dashboard for log monitoring")
