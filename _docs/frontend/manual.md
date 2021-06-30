---
title: Manual do Utilizador
category: Frontend
order: 1
type: 3
---

Destinada aos clientes do SkyView, a plataforma desenvolvida expõe alguns dos *endpoints* disponibilizados pela API, criando assim funcionalidades para os destinatários. Nesta, qualquer utilizador da mesma pode visualizar os voos que sobrevoam a Alemanha em tempo real, o índice de qualidade do ar nos aeroportos, entre outros.

## Maps

Na página denominada "Maps", é possível visualizar um mapa-múndi centrado na Alemanha. Em cima deste, existe um pequeno *switch* que permite ao utilizador escolher um dos dois modos de visualização possíveis nesta página, com as nomenclaturas "Environment's Map" e "Flights' Map" à sua esquerda e direita, respetivamente.

<img src="/images/frontend/worldmap.png" alt="Mapa-Múndi" width="750" height="500">{: .center-image}
*Exemplo da página Maps na plataforma.*

### Flights' Map

Por omissão, o modo exibido será o dos voos, com o interruptor a ser preenchido por um tom de azul. Neste, o cliente pode acompanhar os aviões que sobrevoam a Alemanha em tempo real, com um pequeno ícone de um avião azulado a simbolizar cada voo. Ao longo do tempo, o ícone deslocar-se-á sobre o território alemão, replicando assim a trajetória do voo enquanto este estiver sobre o país.

<img src="/images/frontend/flightsmap.png" alt="Flights Map" width="750" height="500">{: .center-image}
*Exemplo do mapa no modo "Flights' Map.*

Clicando com o botão do lado esquerdo do rato sobre qualquer avião apresentado no mapa, uma nova secção será exibida. Na parte superior desta, o código ICAO 24 correspondente ao avião selecionado será apresentado, bem como dois gráficos.

São estes os gráficos de velocidade (m/s) e de altitude (metros). Tal como o nome indica, o primeiro disponibiliza algumas das velocidades do avião registadas pela aplicação desde que teve conhecimento do mesmo. Ordenando estes valores cronologicamente, um gráfico fiel ao comportamento do avião é estabelecido. O mesmo se passa no segundo caso, registando as diferentes altitudes a que se encontrava o voo ao longo do tempo.  

<img src="/images/frontend/flightspopup.png" alt="Flights Popup" width="750" height="500">{: .center-image}
*Exemplo do que é apresentado ao clicar num avião.*

### Environment's Map

Se o utilizador clicar com o botão esquerdo do rato sobre o *switch*, este alterará o modo em exibição para a vista do ambiente. Esta mudança será acompanhada por uma troca de cor no *switch* de azul para um tom esverdeado. Os ícones representativos de cada voo serão removidos, dando lugar a outro conjunto de símbolos representando os aeroportos alemães.

<img src="/images/frontend/environmentsmap.png" alt="Environments Map" width="750" height="500">{: .center-image}
*Exemplo do mapa no modo "Environment's Map.*

Clicando com o botão do lado esquerdo do rato sobre qualquer ícone apresentado no mapa, uma nova secção será exibida. Nesta, é possível visualizar três parâmetros relativos ao aeroporto selecionado, sendo estes:

1. Airport
1. ICAO
1. Air Quality Index

O primeiro item da lista, apresenta o nome do aeroporto em causa. Já o código aeroportuário ICAO, é utilizado para designar o aeroporto internacionalmente, fazendo parte de um acordo que conta com 180 países membros. O terceiro e último elemento, informa o cliente do índice de qualidade do ar, naquele momento, no aeródromo.

<img src="/images/frontend/environmentspopup.png" alt="Environments Popup" width="750" height="500">{: .center-image}
*Exemplo do que é apresentado ao clicar num aeroporto.*

## Table

Na página denominada "Table", é possível visualizar uma tabela denominada "Incoming Airplanes". Esta tabela será preenchida pelos voos, com destino a um aeroporto alemão, prestes a aterrar.

A tabela está então dividida em quatro colunas:

1. Icao24
1. Avião - Callsign
1. Aeroporto de chegada
1. Distância

Tal como referido na secção [Flight's Map](#flights-map), o código ICAO 24 representa o avião em si, sendo que o *Callsign* é também um identificador único, cujo prefixo corresponde ao seu país de origem. Os últimos dois campos são, tal como indicam, o nome do aeroporto alemão em que o avião está prestes a aterrar e a distância a que se encontra do mesmo.

<img src="/images/frontend/table.png" alt="Tabela" width="750" height="500">{: .center-image}
*Exemplo da página Table na plataforma.*