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
    depends_on: postgres
```
> [!NOTE]
> Criei o arquivo de forma crua e simples apenas para fins de estudo, mas como boa prática é interessante criar um volume para persistência de dados do banco, especificar as variáveis de ambiente em um arquivo `.env` separado.

* Em seguida acesse o `localhost:80` e insira as credenciais definidas no `compose.yaml`:

![image](https://github.com/user-attachments/assets/f5a8098f-7d2f-496b-a240-08260b82b813)

* Dentro do pgadmin, clique em `add server`, na aba `general `preencha o campo `name` como preferir, na aba `connection` preencha `host name/adress` com o nome do container do banco de dados, e os campos de usuário e senha com os valores definidos no `compose.yaml`.

![image](https://github.com/user-attachments/assets/9c049ea7-8e5d-4523-b206-7ddb912cbfbb)
#




