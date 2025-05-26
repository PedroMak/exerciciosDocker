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
### 5.

