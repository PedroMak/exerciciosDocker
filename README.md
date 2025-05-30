# PB Compass UOL - Exercícios de Docker  [![My Skills](https://skillicons.dev/icons?i=docker)](https://skillicons.dev)

### 1. Crie um arquivo Dockerfile que utilize a imagem alpine como base e imprima a mensagem "Olá, Docker!" ao ser executada. Construa a imagem com o nome meu-echo e execute um container a partir dela.

* Dentro de uma dada pasta crie o arquivo Dockerfile com o seguinte conteúdo:

```
FROM alpine:3.21
CMD ["echo", "Hi Docker!"]
```
 * Em seguida, no promt de comando, entre na pasta onde o arquivo está localizado e construa a imagem utilizando o seguinte comando:

```
docker build -t meu-echo .
```

* Após a construção da imagem, execute o container com o comando:

```
docker run meu-echo
```
* Obtendo, assim, a seguinte saída: </br>

![image](https://github.com/user-attachments/assets/3f7c760c-0974-46a4-a2c8-33e754d55071)
#
### 2. Crie um container com Nginx que sirva uma página HTML customizada (index.html). Monte um volume local com esse arquivo para que ele apareça na raiz do site (/usr/share/nginx/html). Acesse a página via http://localhost.

* Primeiro crie um arquivo Dockerfile com o seguinte conteúdo:

```
FROM nginx:stable-alpine3.21-perl
EXPOSE 80
```
* Em seguida crie um volume com o seguinte comando:

```
docker volume create meu-volume
```
* Depois construa a imagem com o comando:

```
docker build -t ex2:pb .
```
* Depois disso rode o container associando o volume a ele:

```
 docker run -d -p 80:80 --name meu-site -v meu-volume:/usr/share/nginx/html ex2:pb
```
* Depois de rodar o container use o comando `docker exec -it meu-site` para entrar no container, entre no diretório `/usr/share/nginx/html`, apague o `index.html` nele existente e crie um novo com o conteúdo que desejar.

![image](https://github.com/user-attachments/assets/75a59711-d3f2-4e11-86cc-4155e6148078)

* Para testar a persistência do volume, rode `docker rm -f meu-site` para remover o container, em seguida rode um novo com `docker run -d -p 80:80 --name meu-site2 -v meu-volume:/usr/share/nginx/html ex2:pb`. Ao acessar o localhost:80 a página customizada deverá estar sendo servida.
#
### 3. Inicie um container da imagem ubuntu com um terminal interativo (bash). Navegue pelo sistema de arquivos e instale o pacote curl utilizando apt.

* Primeiro crie um arquivo Dockerfile com o seguinte conteúdo:

```
FROM ubuntu:24.04
```
* Depois construa a imagem:

```
docker build -t ex3:pb
```
* Em seguida rode o container em modo interativo com o seguinte comando:

```
docker run -it --name exerc3 ex3:pb bash
```
* Dentro do container rode `apt update`, em seguida `apt install -y curl`. Depois da instalação verifique com `curl --version`

![image](https://github.com/user-attachments/assets/4b6f237e-155f-4a53-aa74-2e6c82bd70a2)
#
### 4. Suba um container do MySQL (pode usar a imagem mysql:5.7), utilizando um volume nomeado para armazenar os dados. Crie um banco de dados, pare o container, suba novamente e verifique se os dados persistem.

* Primeiro crie um arquivo Dockerfile com o seguinte conteúdo:

```
FROM mysql:5.7
ENV MYSQL_ROOT_PASSWORD=<senha aqui>
ENV MYSQL_DATABASE=ex4
```
* Depois construa a imagem:

```
docker build -t ex4:pb
```
* Em seguida crie um volume:

```
docker volume create meu-volumeSQL
```

* Rode então o container com o seguinte comando:

```
docker run -d --name exerc4 -v meu-volumeSQL:/var/lib/mysql -p 3306:3306 ex4:pb
```
* Em seguida utilize o seguinte comando para entrar no container:

```
docker exec -it exerc4 mysql -u root -p
```
> [!NOTE]
> Após esse comando, o prompt irá solicitar a senha que foi definida no Dockerfile.

* Dentro do container crie uma tabela e insira informações:

```
USE ex4

CREATE TABLE tecnologias (
  id INT PRIMARY KEY AUTO_INCREMENT,
  nome VARCHAR(100)
  );

INSERT INTO tecnologias (nome) VALUES 
  ('Linux'),
  ('Docker'),
  ('AWS'),
  ('Kubernetes');
```
* Podemos conferir o conteúdo da tabela com `SELECT * FROM tecnologias`.

![image](https://github.com/user-attachments/assets/7377aef7-08c5-4052-865f-e68d9d659733)

* Para conferir a persistência dos dados, digite `exit` para sair do container e remova-o com `docker rm -f exerc4`. Em seguida rode um novo container com `docker run -d --name exerc4.1 -v meu-volumeSQL:/var/lib/mysql -p 3306:3306 ex4:pb`. Digite então `USE ex4` e execute o comando `SHOW TABLES` ou `SELECT * FROM tecnologias` para conferir.

![image](https://github.com/user-attachments/assets/df955993-6686-4bdf-9851-eb55695e8641)
![image](https://github.com/user-attachments/assets/10ffe9c9-8430-4a79-abe8-29ec80f9f100)
#
### 5. Crie um container com a imagem alpine passando uma variável de ambiente chamada MEU_NOME com seu nome. Execute o container e imprima o valor da variável com o comando echo.

* Primeiro crie um arquivo Dockerfile com o seguinte conteúdo:

```
FROM alpine
ENV MEU_NOME=Pedro
```
* Em seguida construa a imagem:

```
docker build -t ex5:pb .
```
* Depois rode o container com o comando:

```
docker run -d -it --name exerc5 ex5:pb
```
* Dentro do container execute o seguinte comando para poder interagir dentro do container:

```
docker exec -it exerc5 sh
```
* Para finalizar, execute `echo Hi $MEU_NOME` para conferir o uso da variável de ambiente.

![image](https://github.com/user-attachments/assets/e5d7a823-9188-4f18-afb2-4653a62ee7b5)
#
### 6. Utilize um multi-stage build para otimizar uma aplicação Go, reduzindo o tamanho da imagem final. Utilize para praticar o projeto [GS PING](https://github.com/docker/docker-gs-ping) desenvolvido em Golang.

* Primeiro baixe a aplicação contida no repositório que está linkado no enunciado e altere o nome do Dockerfile original para que possa construir o seu.
* Em seguida crie seu próprio Dockerfile com o seguinte conteúdo:

```
FROM golang:1.19 AS build-stage
WORKDIR /app
COPY . .

# Remove a necessidade de bibliotecas C do sistema, garante que o binário seja para Linux e gera o binário
RUN CGO_ENABLED=0 GOOS=linux go build -o /docker-gs-ping

FROM gcr.io/distroless/base-debian11 AS build-stage-2
WORKDIR /

# Pega o binário gerado no build anterior
COPY --from=build-stage /docker-gs-ping /docker-gs-ping
EXPOSE 8080

# Executa o binário
ENTRYPOINT ["/docker-gs-ping"]
```
* Depois, construa a imagem:

```
docker build -t ex6:pb .
```
* Para finalizar, rode o comando `docker run --name exerc6 -p 8080:8080 ex6:pb` e visualize no navegador pelo localhost.

![image](https://github.com/user-attachments/assets/e0056a6d-ccec-4d37-a83b-f316795760d9)
* Agora confira o tamanho da imagem original com a imagem criada com seu próprio Dockerfile e irá notar uma grande diferença de tamanho:

![image](https://github.com/user-attachments/assets/c05fbea1-aa82-4202-badd-34483c1acfeb)

#
### 7. Crie um projeto no docker compose para executar o projeto [React Express + Mongo](https://github.com/docker/awesome-compose/tree/master/react-express-mongodb).
#
### 8. Utilize Docker Compose para configurar uma aplicação com um banco de dados PostgreSQL, use para isso o projeto [pgadmin](https://github.com/docker/awesome-compose/tree/master/postgresql-pgadmin).

* Primeiro crie um arquivo compose.yaml e insira as seguintes informações:

```
services:

  postgres:
    image: postgres:13-alpine3.20
    ports: 
      - 5432:5432
    environment: 
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password

  pgadmin:
    image: dpage/pgadmin4
    ports:
      - 80:80
    environment:
      - PGADMIN_DEFAULT_EMAIL=e@mail.com
      - PGADMIN_DEFAULT_PASSWORD=password
    restart: always
    depends_on: 
      - postgres
```
> [!NOTE]
> Criei o arquivo de forma crua e simples apenas para fins de estudo, mas como boa prática é interessante criar um volume para persistência de dados do banco, especificar as variáveis de ambiente em um arquivo `.env` separado.

* Em seguida acesse o `localhost:80` e insira as credenciais definidas no `compose.yaml`:

![image](https://github.com/user-attachments/assets/f5a8098f-7d2f-496b-a240-08260b82b813)

* Dentro do pgadmin, clique em `add server`, na aba `general `preencha o campo `name` como preferir, na aba `connection` preencha `host name/address` com o nome do container do banco de dados, e os campos de usuário e senha com os valores definidos no `compose.yaml`.

![image](https://github.com/user-attachments/assets/9c049ea7-8e5d-4523-b206-7ddb912cbfbb)
#
### 9. Construa uma imagem baseada no Nginx ou Apache, adicionando um site HTML/CSS estático. Utilize a [landing page do Creative Tim](https://github.com/creativetimofficial/material-kit) para criar uma página moderna hospedada no container.

* Primeiro baixe o repositório linkado no enunciado e, dentro da pasta `material-kit-master` crie um Dockerfile com o seguinte conteúdo:

```
FROM nginx:stable-alpine
COPY . /usr/share/nginx/html
EXPOSE 80
```
* Em seguida construa a imagem com o seguinte comando:

```
docker build -t ex9:pb .
```
* Após a construção da imagem, rode o container com o comando:

```
docker run -d --name exerc9 -p 80:80 ex9:pb
```
* Para finalizar, acesse `localhost:80` e confira a página no ar:

![image](https://github.com/user-attachments/assets/fa8bac8d-ecd6-4144-8f82-e87907a73cad)
#
### 10. Ao rodar containers com o usuário root, você expõe seu sistema a riscos maiores em caso de comprometimento. Neste exercício, você deverá criar um Dockerfile para uma aplicação simples (como um script Python ou um servidor Node.js) e configurar a imagem para rodar com um usuário não-root.

* Primeiro crie um Dockerfile com o seguinte conteúdo:

```
FROM node:alpine
RUN adduser -D user
USER user
```
* Em seguida construa a imagem com:

```
docker build -t ex10:pb .
```
* Depois rode o container com:

```
docker run -d -it --name exerc10 ex10:pb
```
* Para finalizar, rode o comando `docker exec -it exerc10 sh` para entrar no container em um shell e rode o comando `whoami` para conferir seu usuário:

![image](https://github.com/user-attachments/assets/140248de-74ed-4911-9e81-5e8317731704)
#
### 11. Trivy é uma ferramenta open source para análise de vulnerabilidades em imagens Docker. Neste exercício, você irá analisar uma imagem pública, como python:3.9, em busca de vulnerabilidades conhecidas.

#### Etapa 1 - instalação do trivy:
* Primeiro é preciso instalar o trivy em sua máquina. Para Windows, entre no repositório da [Aquasecurity](https://github.com/aquasecurity/trivy/releases/tag/v0.62.1) e baixe o arquivo `trivy_0.62.1_windows-64bit.zip`;
* Após extrair o arquivo é preciso adicionar ao Path do Windows para que o trivy possa ser executado de qualquer terminal, para isso, siga o caminho `Painel de Controle → Sistema e Segurança → Sistema → Configurações avançadas do sistema`;
* Na aba `Avançado` clique em `Variáveis de ambiente` e selecione `Path` em `Variáveis de sistema`;
* Clique em `Novo` e adicione o caminho da pasta onde se encontra o `trivy.exe`.

#### Etapa 2 - análise da imagem:
* É preciso ter a imagem desejada em sua máquina local e, para isso, pode-se criar um Dockerfile com o conteúdo `FROM python:3.9` e depois construir a imagem com `docker build -t ex11:pb .`; ou pode-se usar o comando `docker pull python:3.9` em seu terminal para baixá-la diretamente do DockerHub.
* Em seguida rode o comando `trivy imagem <nome_imagem>` para analisar a imagem.

#### Etapa 3 - identificação de vulnerabilidades:
* Ao rodar `trivy imagem <nome_imagem>` teremos a seguinte saída:

![image](https://github.com/user-attachments/assets/cb69ca18-d971-449e-b78b-a57d001afcf9)
* Conseguimos então visualizar a presença de 4 vulnerabilidades, sendo uma de nível médio no `pip` (sistema de gerenciamento de pacotes de python, e três de nível alto em módulos do pacote `setuptools`
* Também foi possível visualizar a presença de mais de 1300 vulnerabilidades na versão Debian em que a imagem se baseia ([aqui](https://raw.githubusercontent.com/PedroMak/exerciciosDocker/refs/heads/main/ex11/vulnerabilidades.json))

#### Etapa 4 - sugestão de melhorias:
* Uma primeira sugestão é utilizar uma imagem python de melhor qualidade. Recomendo buscar imagens mais recentes no DockerHUB, levando em consideração a aba de vulnerabilidades que o site disponibiliza.
* Outra sugestão é atualizar o pacote `setuptools` para uma versão mais recente na qual essas vulnerabilidades já tenham sido resolvidas; versões essas que a saída do trivy já disponibiliza.

![image](https://github.com/user-attachments/assets/31f947e2-d9d9-482c-bfc0-ecdee03abc16)
#
### 12. Após identificar vulnerabilidades com ferramentas como o Trivy, o próximo passo é corrigi-las. Imagens grandes e genéricas frequentemente trazem bibliotecas desnecessárias e vulneráveis, além de usarem o usuário root por padrão. Neste exercício, você irá trabalhar com um exemplo de Dockerfile com más práticas e aplicar melhorias para construir uma imagem mais segura e enxuta. Identifique as melhorias e gere uma nova versão de Dockerfile.

* Para este exercício foram disponibilizados três arquivos:

  * Dockerfile:
  
  ```
  FROM python:3.9
  WORKDIR /app
  COPY requirements.txt .
  RUN pip install -r requirements.txt
  COPY . .
  CMD ["python", "app.py"]
  ```
  * requirements.txt:
  
  ```
  flask==1.1.1
  ```
  * app.py:
 
  ```
  from flask import Flask

  app = Flask(__name__)

  @app.route("/")
  def hello_world(): 
      return "<p>Hello, World!</p>"
  ```
* De cara já podemos identificar a má prática de rodar a aplicação com o `root`, mas antes de alterar o Dockerfile eu a construí com `docker build -t imagem:vul .` e rodei `trivy image imagem:vul` e tive a seguinte saída:

![image](https://github.com/user-attachments/assets/2676c9dd-328a-4ed1-b5b5-7de00fd6a021)

* Conseguimos visualizar a presença de 5 vulnerabilidades, sendo 4 delas de nível alto. Tais vulnerabilidades podem ser resolvidas com o uso de uma imagem mais recente, uma imagem menor (como imagens baseadas em alpine), ou atualizando seu arquivo de dependências para que versões mais recentes sejam instaladas.
* Ao tentar rodar o container com `docker run --name pythonvul -p 5000:5000 imagem:vul` descobri uma série de problemas com a forma que essa aplicação foi estruturada:
  * Ao especificar a versão 1.1.1 do Flask no requirements.txt, o pip baixa essa versão antiga de Flask, mas baixa versões atualizadas de outras dependências, gerando conflitos na execução;
  * O primeiro conflito foi com o `jinja` (um mecanismo de modelo web) que, para consertar, precisei adicionar `jinja2<3.0` ao requirements, para que o pip baixe uma versão antiga e compatível com a versão de Flask utilizada;
  * O segundo conflito foi com `markupsafe` (biblioteca para escape de caracteres), que foi consertado com a adição de `markupsafe<2.1` (versão compatível) ao requiriments;
  * O terceiro conflito foi com `itsdangerous` (biblioteca para transferêcia de dados por ambientes não confiáveis), que foi consertado com a adição de `itsdangerous<2.1` (versão compatível) ao requirements;
  * O quarto, e último, conflito foi com o `werkzeug` (biblioteca que oferece funcionalidades que facilitam o desenvolvimento web), que foi consertado com adição de `werkzeug<2.1` (versão compatível) ao requirements;
* Após essas correções, ao rodar o comando `docker run --name pythonvul -p 5000:5000 imagem:vul` ele apresentou um erro, no qual a aplicação não iniciou e o container fechou instantaneamente, para corrigir esse erro foi necessário alterar a linha `CMD ["python", "app.py"]` para `CMD ["python", "-m", "flask", "run", "--host=0.0.0.0"]`
  * Dentro de `app.py` não há nada que inicie o servidor Flask, então rodar o script python com `CMD ["python", "app.py"]` não funciona;
  * Já com `CMD ["python", "-m", "flask", "run", "--host=0.0.0.0"]` você está usando o módulo Flask para iniciar o servidor, sendo necessário também especificar `--host-0.0.0.0` para que possa ser acessado fora do container.
* Após essas correções conseguimos rodar o container e acessá-lo no localhost:

![image](https://github.com/user-attachments/assets/44bc928e-6d84-4ec5-8dc6-12c7e0a20478)

* Agora, para melhorar o Dockerfile podemos adicionar um usuário para que não rode como `root`, podemos escolher uma imagem mais leve como `python:3.9-slim` e podemos fazer um multi-stage para evitar que arquivos desnecessários façam parte da imagem.
* Após essas alterações nossos arquivos se encontram assim:

  * Dockerfile:
    
  ```
  FROM python:3.9 AS builder
  WORKDIR /app
  COPY requirements.txt .
  RUN pip install --prefix=/dependencies -r requirements.txt

  FROM python:3.9-slim
  RUN useradd -m user
  WORKDIR /
  COPY --from=builder /dependencies /usr/local
  COPY app.py .
  EXPOSE 5000
  USER user
  CMD ["python", "-m", "flask", "run", "--host=0.0.0.0"]
  ```
  * requirements.txt:
    
  ```
  flask==1.1.1
  jinja2<3.0
  markupsafe<2.1
  itsdangerous<2.1
  werkzeug<2.1
  ```
> [!NOTE]
> `--prefix=` usado no Dockefile serve para instalar as dependências em uma pasta diferente da padrão.

* Após essas melhorias podemos conferir a redução de tamanho da imagem e que o container está rodando com o usuário criado via Dockerfile:
 ![image](https://github.com/user-attachments/assets/37abf3fa-bc70-4770-b970-576734c0d5b6)
 ![image](https://github.com/user-attachments/assets/4809f940-16c7-4b3e-b5a1-ebc86730a614)

* Minha recomendação final seria atualizar as dependências para versões mais recentes onde as vulnerabilidades listadas pelo trivy foram consertadas.
#
### 13. Crie um Dockerfile que use a imagem python:3.11-slim, copie um script Python local (app.py) e o execute com CMD. O script pode imprimir a data e hora atual. Faça o push da imagem para o Docker Hub.

* Para este exercício é necessário uma conta no [DockerHub](https://hub.docker.com/), que pode ser facilmente criada clicando em `sign up` na página principal, que irá abrir a seguinte tela:

![image](https://github.com/user-attachments/assets/c8a0fa77-64f1-4d5f-8cd5-853e3d1ac2bb)

* Vá na aba de repositórios e crie um:

![image](https://github.com/user-attachments/assets/d5e2c26b-a593-4192-ac30-6cfc28185e25)

* Em seguida crie o seguinde script em python para imprimir a data e a hora atual:

```
import datetime

data = datetime.datetime.now()

print("Dia:", data.strftime("%d"), "/", data.strftime("%m"), "/", data.strftime("%Y"))
print("Hora:", data.strftime("%H"), ":", data.strftime("%M"))
```
* Depois crie o seguinte Dockerfile:

```
FROM python:3.11-slim
COPY . .
CMD ["python", "app.py"]
```
* Construa então a imagem com o comando `docker build -t meu-echo:pb .` e depois rode o container com `docker run --name meu-app meu-echo:pb` que nos dará a seguinte saída:

![image](https://github.com/user-attachments/assets/a1b82d52-ebdb-4d98-a24a-a40f6297b72f)

* Agora, para enviarmos nossa imagem para o DOckerHub é necessário que alteremos o nome da imagem para que fique compatível com o nome do repsitório, fazemos isso com o seguinte comando:

```
docker image tag meu-echo:pb pedromak/exercicio13:v1
```
* Em seguida executamos `docker login` para nos autenticarmos:

![image](https://github.com/user-attachments/assets/a53c1208-0834-480a-8651-a85b9b8cc653)

> [!NOTE]
> Caso seja a primeira vez será solicitado seu usuário e senha.

* Para finalizar executamos `docker push pedromak/exercicio13:v1` e podemos conferir no DockerHub que nossa imagem estará lá:

![image](https://github.com/user-attachments/assets/dd9e3afa-82d5-4dcf-b316-04b1e078afb1)

