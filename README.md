# Exercícios de Docker - PB Compass UOL

### 1. Crie um arquivo Dockerfile que utilize a imagem alpine como base e imprima a mensagem "Olá, Docker!" ao ser executada. Construa a imagem com o nome meu-echo e execute um container a partir dela.

* Dentro de uma dada pasta criei o arqueivo Dockerfile com o seguinte conteúdo:

```
FROM alpine:3.21
CMD ["echo", "Hi Docker!"]
```
 * Em seguida, no promt de comando, entrei na pasta onde o arquivo está localizado e construí a imagem utilizando o seguinte comando:

```
docker build -t meu-echo .
```

* Após a construção da imagem, executei o container com o comando:

```
docker run meu-echo
```

* Obtendo, assim, a seguinte saída: </br>

![image](https://github.com/user-attachments/assets/3f7c760c-0974-46a4-a2c8-33e754d55071)

### 2. Crie um container com Nginx que sirva uma página HTML customizada (index.html). Monte um volume local com esse arquivo para que ele apareça na raiz do site (/usr/share/nginx/html). Acesse a página via http://localhost.
