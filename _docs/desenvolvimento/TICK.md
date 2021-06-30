---
title: Métricas
category: Operações
order: 4
type: 4
---

Em termos da obtenção periódica e persistência centralizada de valores de métricas de estado atual e desempenho dos diversos componentes e micro-serviços foi utilizado a TICK stack, dada a sua abertura, expansibilidade e facilidade de integração e de utilização.
A instanciação e interligação dos seus componentes constituintes (Telegraf, InfluxDB, Chronograf, e Kapacitor) é assegurada através do uso de Docker-Compose.

![Alt text](https://lh5.googleusercontent.com/FcW4kuGJcVRBmOFPyBmaIsLypBkmh2UAcqRTFp0IwNfj1nlHotmClBMoQPyge8ZvL-YGaHnlXxr6d_b64DHdOkHGaXvx4pLPAGIg14HGgYaSb-rV_1PgO6x63LRRGzdC3w
 "Title")

A recolha de métricas é efetuada periodicamente de cada um dos consumers desenvolvidos, através da integração nativa Quarkus Micrometer.
Desta forma, são automaticamente incluídos diversos data points acerca do estado atual e desempenho da JVM, do módulo de integração com o Kafka (consumer e producer), e do módulo RESTEasy, bem como as métricas customizadas desenvolvidas pela equipa no próprio código fonte.

Estas incluem contadores de número de pedidos REST recebidos, tamanho das listas de vôos/aeroportos recebidos dos tópicos Kafka, número de inserções no InfluxDB, número de eventos gerados, entre outros.

A visualização do estado atual dos vários componentes do sistema at-a-glance é possibilitada por um dashboard Cronograf criado para o efeito, e que inclui um conjunto não exaustivo de métricas cruciais.

![Alt text](/images/posts/es_tick.png?raw=true "Title")

- System CPU Usage
- System Load Average
- System Memory Usage
- Events generated per 10 seconds ('events' kafka topic)
- InfluxDB write operations executed per 10 seconds
- Kafka messages consumed per 10 seconds
- Kafka messages produced per 10 seconds
- /stream SSE requests per 10 seconds
- /all REST requests per 10 seconds
- OpenWeather API response list size
- OpenFlight API response list size