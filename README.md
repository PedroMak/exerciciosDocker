# Exercícios de Docker - PB Compass UOL

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

