---
title: Perspetiva do Programador
category: Frontend
order: 2
type: 3
---

Nesta página será feito um resumo do trabalho efetuado na plataforma desenvolvida para o cliente.

Tal como é possível visualizar no gráfico referente à Arquitetura, esta *web app* resulta de um conjunto de páginas concebidas utilizando a *framework* [React](https://reactjs.org).

## Maps

Na página denominada "Maps", vários tipos de mapas foram experimentados pela equipa, acabando por recorrer a um mapa disponibilizado pela ferramenta [OpenStreetMap](https://www.openstreetmap.org).

### Flights' Map

A lista de voos que sobrevoam a Alemanha em tempo real é obtida através do *endpoint*: /flights/stream. De notar que, devido ao vasto tráfego que geralmente sobrevoa a Alemanha, esta lista foi filtrada para apenas 20 voos, evitando assim *spam* visual e permitindo ao cliente aceder com mais facilidade a um determinado ícone. O código ICAO 24 bem como as coordenadas que permitem colocar o ícone sobre o mapa são então retiradas deste *endpoint*.

Os aviões exibidos são colocados nas coordenadas recebidas, sendo que ao clicar sobre um dos ícones, um segundo *endpoint* é chamado: /flights/all?icao24=<id>. Assim, é possível obter-se com maior detalhe informações sobre um voo em específico, sendo este filtrado pelo seu código ICAO 24. Aqui, a altitude geográfica, velocidade e *timestamp* associada a estes é obtida, providenciando assim todos os constituintes necessários para elaborar os gráficos que o cliente vê quando clica sobre o ícone.

### Environment's Map

Os dados referentes ao índice de qualidade do ar nos aeroportos alemães são obtidos através do *endpoint*: /environment/stream. O código ICAO do aeródromo, o seu nome, localização em coordenadas e índice de qualidade do ar são então retiradas deste *endpoint*.

Os aeroportos exibidos são colocados nas coordenadas recebidas, sendo que ao clicar sobre um dos ícones, os restantes parâmetros tornam-se visíveis. Assim, é possível obter-se com maior detalhe informações sobre um aeroporto em específico quando o cliente clica com o botão do lado esquerdo sobre o ícone.

## Table

Na página denominada "Table", uma tabela foi constituída pela informação gerada no *endpoint*: /arrivals/stream. O código ICAO 24 do voo, o *callsign* do avião, o seu aeroporto de destino bem como a distância a que se encontra do mesmo são então retirados deste *endpoint*.
