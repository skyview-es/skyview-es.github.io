---
title: Jenkins
category: Especificação
order: 1
type: 2
published: true
---

### Overview

No que toca a pipeline **Jenkins** disponibilizada, esta permite que o produto corrente seja sempre continuamente e automaticamente atualizado, e sequencialmente pronto a ser utilizado, garantindo que o software da versão estável mais recente esteja sempre disponível e funcional no ambiente de execução cliente. No que toca ao seu desenho geral, foi delineada uma pipeline standard, que segue o modelo geral **build-test-package-deploy**.

![pipeline-jenkins-ov](/images/spec/es_pipeline_overview.PNG?raw=true "Pipeline Jenkins Overview")

A configuração completa da pipeline não é apenas contemplada no código: na configuração desta pipeline no ambiente Jenkins a trabalhar, deve ser introduzido para usar o Jenkinsfile disponibilizado, e ainda deve ser dado corretamente o repositório na qual se encontra este projeto, de forma de haver um *checkout* implícito ao SCM antes de operacionalizar o *workflow* dado. Para isto, de notar que também é necessário configurar credenciais para o acesso ao repositório SCM pretendido.

### Fases da pipeline

As **fases de execução** da presente pipeline são as seguintes:

- **Build Java 8** – Faz a assemblagem de cada módulo Java 8, utilizando o Maven. Os testes não podem ser executados nesta fase, pois serão realizados numa fase posterior. Devido a haver seis módulos independentes Java no nosso projeto, e sendo que era desejado que o código da pipeline fosse o mais curto possível e o mais programático possível (no caso da quantidade de módulos aumentasse no futuro, e assim ser mais rápido de integrar um novo módulo em todas o ciclo de vida da pipeline), usou-se uma script que passa por uma lista de nomes de diretórios presentes no projeto, com o código de cada módulo, na qual executa a diretiva **“install”** do Maven, com a propriedade **“skipTests”**.

- **Build Java 11** – Similarmente à fase anterior, é feito o *build* de cada módulo Java, neste caso com a versão 11. Embora faça parte do mesmo contexto do que seria normalmente uma fase, é necessária esta divisão devido à diretiva para utilização do Java 11 nesta fase. Esta maneira foi a mais fácil de ser integrada na pipeline.

- **Test Java 8** – Esta fase trata de executar todos os testes de integração ou unitários em todos os módulos Java do software. Também é executado de forma programática e sequencial para cada aplicação Java 8, portanto detém o mesmo fluxo é feito para as fases anteriores. A diretiva de *lifecycle* Maven é a utilizada para utilizar todos os testes, que é o **“test”**. 

- **Test Java 11** – Com semelhanças às fases de build explicitadas anteriormente, este usa também a diretiva **“tools”** com a configuração da versão de SDK para Java 11. Esta fase, tal como a anterior, faz sequencialmente os testes aos módulos Java dos diretórios dos micro-serviços Java 11, incluídos em cada um destes *(./src/test/\*)*.
```
tools {
    jdk "jdk11"
}
```

- **Upload artifacts** – Após as fases lógicas de *build* e teste, esta fase assegura que os artefactos resultantes da assemblagem dos pacotes de software (JAR/WAR) da versão correspondente, seja armazenado num repositório de artefactos JFrog. Neste caso, o repositório é o oficial da disciplina, em **http://192.168.160.49:8081/artifactory/**maven_es2021. Isto é configurado no pom.xml e no settings.xml, e é usado apenas para o upload destes artefactos. Esta fase é transparente à versão do Java utilizada.

- **Push images** – Esta fase realiza o *build* das imagens Docker e, consequentemente, o upload das imagens resultantes para o Docker Registry, que vai ser utilizado pela máquina de instalação do produto como a fonte de imagens no docker-compose de instalação. Neste caso, o repositório utilizado é, também, o oficial, localizado em **http://192.168.160.48:5000**. Esta fase faz o *build* a partir dos Dockerfiles localizados em cada módulo, e upload sequencial (*push*) para cada módulo do projeto. A fase inclui todos os componentes, **incluindo a aplicação Node/ReactJS**.

- **Deploy** – Esta fase acede à máquina de instalação para a entrega do pacote de software, mediante credenciais. Nesta versão final do projeto, a instalação de todos os serviços é feita numa máquina gerida pelo grupo, acedida em **skyview.rrosmaninho.com, no porto 443**, usando credenciais configuradas com um utilizador dependente (**‘jenkins’**) para o efeito. 
Esta fase acede primeiramente à máquina e limpa a instalação anterior, removendo todos os seus componentes. De seguida, remove ficheiros dependentes, como o manisfest YAML de instalação automática dos variados serviços (*docker-compose.yml*), assim como ficheiros de configuração de alguns serviços, necessários para o seu correto funcionamento. Depois, são novamente copiados para a mesma máquina o manifest e os ficheiros de configuração atualizados. Por fim, faz a execução do YAML sobre o Docker daemon (a partir do ```docker-compose up```).

### Alterações posteriores

No caso de a máquina remota poder ser diferente ou na eventualidade da sua migração para outra localização, a definições dos campos da variável global **"remote"** devem ser atualizados: o *“host”* e o *“port”*. O campo *“name”* também pode ser mudado, no entanto não é fundamental para o funcionamento dos acessos SSH, servindo apenas para fazer a identificação posterior. 

Esta máquina destino para a instalação e entrega do produto **deve ter um Docker deamon preparado com execução de comandos em modo *non-root***. Quaisquer certificados essenciais para o acesso ao Docker Registry devem ser instalados na máquina (certificados cliente em */etc/docker/certs*; no caso de haver um certificado de CA Root independente, é ainda necessário instalá-lo no seu sistema Linux; no entanto, se for um ficheiro em formato PEM com a cadeia de certificação, é apenas necessário coloca-lo no diretório dito primeiramente).

No caso de possuir um registry Docker privado, é possível mudá-lo facilmente no código da pipeline, sendo que pode ser alterado na variável de ambiente *REGISTRYHOST* existente. Assim, o *push* e *pull* das imagens Docker na fase de **Upload images** poderá ser feito de outro Registry facilmente que não o por defeito neste projeto. É necessário, no entanto, que este esteja acessível tanto da máquina target, como na ferramenta Jenkins que vai ser utilizada.
