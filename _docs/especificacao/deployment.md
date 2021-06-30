---
title: Deployment
category: Especificação
order: 2
type: 2
published: true
---

## Arquitetura geral

O ambiente de execução e instalação integrado neste projeto assume **um padrão de micro-serviços envolvidos em containers**, com recurso à ferramenta **Docker**, e ainda necessita de suporte para a ferramenta opcional **docker-compose**, acedidas por **modo não-privilegiado**.

A arquitetura é explicada pelo diagrama abaixo. A rede de containers para o efeito é denominada **subnet-proj**, e corresponde à gama de endereços **172.29.0.0/24**. Todos os containers deste projeto situar-se-ão nesta rede Docker.

![docker-comp-deployment-arch](/images/spec/es_docker_arch.png?raw=true "Docker deployment diagram")

É utilizado um ficheiro **docker-compose.yaml** por forma a realizar o *setup* completo de todos os serviços. A arquitetura de instalação providenciada explica o próprio modelo de instalação que segue entre módulos.

## Outros requisitos

Em termos de requisitos especiais, a máquina remota deve estar configurada com o ficheiro-parâmetro de sistema Linux **vm.max_map_count** com o valor mínimo de 262144, devido a ser um **requerimento do Elasticsearch** na versão 7.13.x . 

## Localização

De notar que **todos os serviços do projeto serão instalados na mesma máquina, incluindo bases de dados, brokers e ferramentas de monitorização, logging e visualização**. Isto significa que, no âmbito deste projeto, **não são utilizadas as máquinas virtuais fornecidas** para instalação do produto e acesso aos recursos de dados (partilhados) para as dependências do software desenvolvido – isto deve-se ao facto de não ter sido possível fazer a instalação da *playground VM* disponibilizada para o efeito. Como esta ficou sem recursos disponíveis para todos os projetos rapidamente, o mesmo tipo de problema poderia ter acontecido com a *runtime VM*, então optou-se por um ambiente gerido pelo grupo, na qual estivesse acedido pela Web (**skyview.rrosmaninho.com**).

