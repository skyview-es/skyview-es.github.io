---
title: Producers
category: Desenvolvimento
order: 3
type: 3
published: true
---

## Flights Producer

Flights-producer foi feito com *SpringBoot* e utiliza a API OpenSky, fazendo duas calls distintas e produzindo informação para dois topics diferentes: *flights* e *arrivals*.

### *flights*
Para o preenchimento do topic *flights* é sempre feita a seguinte chamada:

```
https://opensky-network.org/api/states/all?lamin=47.2&lomin=5.5&lamax=55.1&lomax=15.5
```

O resultado tem todos os voos cuja latitude e longitude estão delimitados pelos limites da Alemanha, obtendo assim todos os voos que estão atualmente a sobrevoar a área definida. Este resultado é guardado na classe *FlightsState*.

A mensagem é enviada para o topic em formato *json* através do método *gson.toJson()*.

Exemplo:
```
{
  "time": 1625017570,
  "states": [
    [
      "4bc847",
      "PGT33P  ",
      "Turkey",
      1625017569,
      1625017570,
      11.2141,
      49.8804,
      10675.62,
      false,
      212.37,
      117.25,
      0,
      null,
      10949.94,
      "6426",
      false,
      0
    ],
    ...
}
```

### *arrivals*
Para o preenchimento do topic *arrivals* é sempre feita a seguinte chamada (com begin e end sendo limites temporais em *epoch*):

```
"https://opensky-network.org/api/flights/all?begin="+begin+"&end="+end
```

O resultado tem a lista de todos os voos que estão/estavam ativos (no ar) no limite de tempo definido. Este resultado é guardado na classe *Flights* e depois filtrado para apenas sobrar os voos cujo parametro *estArrivalAirport* tenha um código *ICAO* de um aeroporto alemão. Este parametro é depois alterado para o nome do aeroporto através de um dicionário que relaciona os valores *ICAO* com os nomes reais.

A mensagem é depois enviada para o topic em formato *json* através do método *gson.toJson()*.

Exemplo:
```
[
  {
    "icao24": "48548c",
    "firstSeen": 1558953893,
    "estDepartureAirport": "EHAM",
    "lastSeen": 1558957226,
    "estArrivalAirport": "Stuttgart",
    "callsign": "KLM1873",
    "estDepartureAirportHorizDistance": 1217,
    "estDepartureAirportVertDistance": 94,
    "estArrivalAirportHorizDistance": 1086.0,
    "estArrivalAirportVertDistance": 14.0,
    "departureAirportCandidatesCount": 0,
    "arrivalAirportCandidatesCount": 5
  },
  ...
]
```

## Arrived Producer (and Consumer)
Arrived-producer foi feito com *Quarkus* e consome os dados do topic *arrivals*, produzidos pelo producer anterior, processando-os para depois produzir informação para dois topics diferentes: *arrived* e *events*.

Em termos de processamento é utilizada a lógica de guardar sempre a última lista de voos recebida do topic *arrivals*, para quando consumir uma lista nova, poder comparar e verificar se há alguns voos novos ou se houve algum voo que desapareceu da lista.

Estas listas são de facto objectos da classe *Flights*, obtidas através do método *gson.fromJson()* que recebe o *json* consumido.

### *arrived*
É produzido para o topic *arrived* quando se verifica que houve algum voo (classe *Flight*) que desapareceu da lista (classe *Flights* que contem uma lista de *Flight*).

Assim, é produzida para o topic uma mensagem individual para cada *Flight* que se deteta que aterrou, utilizando o método *gson.toJson()*. 

Exemplo:

```
{
    "icao24": "48548c",
    "firstSeen": 1558953893,
    "estDepartureAirport": "EHAM",
    "lastSeen": 1558957226,
    "estArrivalAirport": "Stuttgart",
    "callsign": "KLM1873",
    "estDepartureAirportHorizDistance": 1217,
    "estDepartureAirportVertDistance": 94,
    "estArrivalAirportHorizDistance": 1086.0,
    "estArrivalAirportVertDistance": 14.0,
    "departureAirportCandidatesCount": 0,
    "arrivalAirportCandidatesCount": 5
}
```

### *events*
É produzido para o topic *events* quando se verifica que apareceu ou desapareceu um voo da lista, com mensagens distintas para cada um.

Exemplo de uma mensagem produzida quando aparece um novo voo na lista:

```
    New flight '48548c' approaching Stuttgart airport at 1086.0 meters.
```

Exemplo de uma mensagem produzida quando desaparece um voo da lista:

```
    Flight with id '48548c' arrived in Stuttgart airport.
```

## Environment Producer

### *environment*
Environment-producer utiliza a API OpenWeatherMap, fazendo dezassete calls distintas e produzindo informação para o topic *environment*.

Primeiramente fomos pesquisar os valores de latitude e longitude para os 17 aeroportos mais frequentados na Alemanha obtendo a seguinte tabela, sendo estes os códigos *ICAO* que também sobram da filtragem no flights-producer.

| ICAO |   LAT  | LON   |
|:----:|:------:|-------|
| EDDB | 52.37  | 13.51 |
| EDDT | 52.55  | 13.28 |
| EDDW | 53.04  |  8.78 |
| EDDK | 50.85  |  7.13 |
| EDLW | 51.51  |  7.61 |
| EDDC | 51.13  | 13.76 |
| EDDL | 51.28  |  6.76 |
| EDDF | 50.03  |  8.57 |
| EDDH | 53.62  |  9.98 |
| EDDV | 52.46  |  9.68 |
| EDSB | 48.77  |  8.08 |
| EDDP | 51.42  | 12.23 |
| EDDM | 48.35  | 11.77 |
| EDDG | 52.13  |  7.68 |
| EDDN | 49.46  | 11.06 |
| ETAR | 49.43  |  7.60 |
| EDDS | 48.68  |  9.21 |

Para obter o index de poluição para cada aeroporto realizamos a seguinte chamada, fornecendo a latitude e longitude previamente guardada:

```
http://api.openweathermap.org/data/2.5/air_pollution?lat="+lat[i]+"&lon="+lon[i]+"&appid="+apikey;
```

Exemplo de um resultado da chamada:

```
{
  "coord": {
    "lon": 13.51,
    "lat": 52.37
  },
  "list": [
    {
      "main": {
        "aqi": 1
      },
      "components": {
        "co": 203.61,
        "no": 0,
        "no2": 10.2,
        "o3": 52.93,
        "so2": 3.93,
        "pm2_5": 4.59,
        "pm10": 4.96,
        "nh3": 0.65
      },
      "dt": 1625022000
    }
  ]
}
```

Este resultado é guardado na classe *ComponentsClass*, e depois é enviado como argumento para o construtor da classe *ComponentsByAirport* que também recebe o código *ICAO* e o nome do aeroporto. 

No fim de todas as chamadas, estes 17 objectos *ComponentsByAirport* são guardados numa lista de *ComponentsByAirport*, definida pela classe *ComponentsGermany*, que desta forma agrupa todos os dados de poluição de todos os aeroportos pretendidos.

Assim, é produzida para o topic uma mensagem em formato *json* utilizando o método *gson.toJson()*.

Exemplo:

```
{
    "airports":[
        {
            "airporticao":"EDDB",
            "airportname":"Berlin-Schönefeld",
            "airportcomponents":
                {"coord":
                    {
                        "lon":13.519,
                        "lat":52.3738
                    },
                    "list":[
                        {
                            "main":
                            {
                                "aqi":1
                            },
                            "components":
                            {
                                "co":203.61,
                                "no":0.0,
                                "no2":10.2,
                                "o3":52.93,
                                "so2":3.93,
                                "pm10":4.96,
                                "nh3":0.65
                            },
                            "dt":1625022000
                        }
                    ]
                }
        },
        ...
    ]
}
```