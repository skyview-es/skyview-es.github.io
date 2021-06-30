---
title: Consumers
category: Desenvolvimento
order: 2
type: 3
---

Os módulos consumer constituem uma parte integral da arquitetura desenvolvida, visto que desempenham as seguintes funções:

- Consumir dados provenientes dos tópicos Kafka populados pelos producers associados às APIs OpenSky e OpenWeather
- Aplicar filtros, transformações, e outras operações aos mesmos
- Ler e Escrever a informação relevante numa base de dados time-series InfluxDB (utilizando Flux)
- Produzir dados processados e eventos para tópicos de Kafka 
- Servir atualizações em real-time para clientes de front-end através do uso de Server Sent Events (com interligação por Reactive Messaging)
- Disponibilizar endpoints REST para obtenção de dados históricos em massa.

![Alt text](/images/posts/es_consumers.png?raw=true "Title")

### Flights-Consumer

A sua função é consumir os dados do endpoint /states da OpenSky API que são disponibilizados no tópico Kakfa 'flights' pelo producer correspondente.
Estes são persistidos no InfluxDB (utilizando o InfluxDBClient) e enviados como Server Sent Events para o frontend e quaisquer outros clientes SSE conectados
Adicionalmente, os dados históricos são disponibilizados pelo endpoint REST /all, que recebe o Query Parameter 'icao24' como filtro e interage com o InfluxDB utilizando Flux para obter os dados. 

Exemplo de payload JSON retornada pelo endpoint /all (com filtro por, neste caso, o icao24 '34310d'):
```
{
  "time": 1625023000,
  "states": [
    {
      "icao24": "34310d",
      "callsign": "BCS3844 ",
      "originCountry": "Spain",
      "timePosition": "2021-06-30T04:16:39",
      "lastContact": 1625022999,
      "longitude": 12.2542,
      "latitude": 51.4113,
      "baroAltitude": 0,
      "onGround": true,
      "velocity": 3.86,
      "trueTrack": 84.38,
      "verticalRate": 0,
      "sensors": "",
      "geoAltitude": 0,
      "squawk": "",
      "spi": false,
      "positionSource": 0
    },
    {
      ...
    }
  ]
}
```

### Arrivals-Consumer

A sua função é consumir os dados do endpoint /flights da OpenSky API que são disponibilizados no tópico Kakfa 'arrivals' pelo producer correspondente.
Estes enviados como Server Sent Events para o frontend e quaisquer outros clientes SSE conectados

Exemplo de payload JSON retornada pelo endpoint SSE /stream:
```
{
  "voos": [
    {
      "icao24": "3cce6f",
      "firstSeen": 1560980170,
      "estDepartureAirport": "EDDS",
      "lastSeen": 1560981639,
      "estArrivalAirport": "Stuttgart",
      "callsign": "FCK211  ",
      "estDepartureAirportHorizDistance": 353,
      "estDepartureAirportVertDistance": 7,
      "estArrivalAirportHorizDistance": 1051,
      "estArrivalAirportVertDistance": 7,
      "departureAirportCandidatesCount": 0,
      "arrivalAirportCandidatesCount": 7
    },
    {
      ...
    }
  ]
}
```

### Environment-Consumer

A sua função é consumir os dados da OpenWeather API que são disponibilizados no tópico Kakfa 'environment' pelo producer correspondente.
Estes são persistidos no InfluxDB (utilizando o InfluxDBClient) e enviados como Server Sent Events para o frontend e quaisquer outros clientes SSE conectados
Adicionalmente, os dados históricos são disponibilizados pelo endpoint REST /all, que recebe o Query Parameter 'icao' como filtro e interage com o InfluxDB utilizando Flux para obter os dados. 

Exemplo de payload JSON retornada pelo endpoint /all (com filtro por, neste caso, o icao '34310d'):
```
[
  {
    "name": "Berlin-Schönefeld",
    "icao": "EDDB",
    "time": {
      "seconds": 1625025577,
      "nanos": 55186000
    },
    "latitude": 52.3738,
    "longitude": 13.519,
    "aqi": 1,
    "co": 211.95,
    "no": 0.01,
    "no2": 12.51,
    "o3": 49.35,
    "so2": 4.77,
    "pm10": 5.2,
    "nh3": 0.74
  },
  {
    ... 
  }
]
```

### Arrived-Consumer

A secção 'Producers' contém os detalhes sobre a função desempenhada por este componente.