# Iniciativa Kubernetes - KubeDev.io

## Comandos

### Aula 1 - Containers e Docker Simplificados

Executar primeiro container
```docker cli
docker container run hello-world
```

Listar containers ativos (em execução)
```docker cli
docker container ls
```

Executar primeiro container, nomeando-o
```docker cli
docker container run --name meuContainer hello-world
```

Remover container usando nome
```docker cli
docker container rm meuContainer
```

Remover container usando CONTAINER ID
```docker cli
docker container rm 487038a500a8
```

Após execução remover o container automaticamente
```docker cli
docker container run --name meuContainer --rm hello-world
```

Subir container no modo interativo, usando SO Ubuntu e abrir terminal bash para interação
```docker cli
docker container run -it ubuntu /bin/bash
```

No terminal aberto do Ubuntu
```shell
apt-get update

apt-get install curl

curl https://www.google.com

exit
```

Subir container nginx (desta forma o terminal fica preso ao output do nginx)
```docker cli
docker container run nginx
```

Subir container nginx em segundo plano, liberando assim o terminal
```docker cli
docker container run -d nginx
```

Subir container nginx com acesso (port binding) à página que o mesmo está servindo  (porta local:porta do container )
```docker cli
docker container run -d -p 8080:80 nginx
```

Remover (forçado) container que está em execução, usando ID do container
```docker cli
docker container rm -f d6ffb2d67f91
```

Subir container mongoDB (precisa passar credenciais, utilizando variáveis de ambiente)
```docker cli
docker container run -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=mongouser -e MONGO_INITDB_ROOT_PASSWORD=mongopwd mongo
```

Remover (forçado) container mongo em execução
```docker cli
docker container rm -f 5f255289bd50
```

#### Comandos de Troubleshooting


Inspecionar o container, usando ID do container
```docker cli
docker container inspect 6e61e8a44de2
```

Execução de um comando no container
```docker cli
docker container exec -it 6e61e8a44de2 /bin/bash
```

Parar execução de um container
```docker cli
docker stop 6e61e8a44de2
```

Iniciar execução de um container
```docker cli
docker start 6e61e8a44de2
```

Ver logs do container
```docker cli
docker container logs 6e61e8a44de2
```

Ver logs do container, filtrando quantidade de logs (linhas)
```docker cli
docker container logs -n 5 6e61e8a44de2
```

Ver logs do container de forma contínua, conforme é gerado novos logs, é exibido na tela
```docker cli
docker container logs -f 6e61e8a44de2
```

Ver logs do container com data e hora
```docker cli
docker container logs -t 6e61e8a44de2
```

#### Comandos para criar imagem Docker

##### Docker Commit (má prática, não recomendado)


Subir container Ubuntu
```docker cli
docker container run -it ubuntu /bin/bash
```

No terminal aberto do Ubuntu
```shell
apt-get update

apt-get install curl --yes

curl

exit
```

Criar nova imagem a partir da imagem executada anteriormente, usando ID e passando nome da imagem a ser criada
```docker cli
docker commit a3a489d41227 ubuntu-curl-commit
```

Listar as imagens
```docker cli
docker image ls
```

Subir container ubuntu-curl-commit, recém criado
```docker cli
docker container run -it ubuntu-curl-commit /bin/bash
```

No terminal aberto do Ubuntu
```shell
curl

curl https://www.google.com

curl

exit
```

Executar container ubuntu-curl-commit já executando curl
```docker cli
docker container run -it ubuntu-curl-commit curl https://www.google.com
```

##### Dockerfile (boa prática, maneira correta)

Criar arquivo Dockerfile com as configurações da imagem a ser criada
```dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install curl --yes
```

Criar imagem docker, passando nome (-t) e contexto (nesse caso é local: .)
```docker cli
docker image build -t ubuntu-curl-file .
```

Verificar imagem criada
```docker cli
docker image ls
```

Adicionar nova configuração ao Dockerfile
```dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install curl --yes
RUN apt-get install vim --yes
```

Gerar novamente a imagem para atualizar com a nova configuração
```docker cli
docker image build -t ubuntu-curl-file .
```

Concatenar as configurações do Dockerfile para evitar uso de cache para atualizar imagem

```dockerfile
FROM ubuntu
RUN apt-get update && apt-get install curl --yes && apt-get install vim --yes
```

Desta forma, ao gerar novamente a imagem, não irá considerar o cache e sempre irá baixar tudo o que for necessário
```docker cli
docker image build -t ubuntu-curl-file .
```

Remover a instrução para instalação do vim no Dockerfile
```dockerfile
FROM ubuntu
RUN apt-get update && apt-get install curl --yes
```

Ao gerar novamente a imagem, não irá considerar o cache e sempre irá baixar tudo o que for necessário
```docker cli
docker image build -t ubuntu-curl-file .
```

Verificar a imagem criada
```
docker image ls
```

Remover imagens sem referência (que não está mais sendo utilizada)
```
docker image prune
```

Remover imagem usando nome
```
docker image rm ubuntu-curl-file
```

Inspecionar imagem
```
docker image inspect 6215c720e7f3
```

Listar histório de criação da imagem, usando nome
```
docker image history ubuntu-curl-commit
```

###### Possibilidades com Dockerfile

FROM => Inicializa o build de uma imagem a partir de uma imagem base

RUN => Executa um comando

LABEL => Adiciona metadados a imagem

CMD => Define o comando e/ou os parâmetros padrão

EXPOSE => Define que o container precisa expor a porta em questão

ARG => Define um argumento pra ser usado no processo de construção

ENV => Define variáveis de ambiente

ADD => Copia arquivos ou diretórios ou arquivos remotos e adiciona ao sistema de arquivos da imagem

COPY => Copia arquivos ou diretórios e adiciona ao sistema de arquivos da Imagem

ENTRYPOINT => Ajuda você a configurar um container que pode ser executado como um executável

VOLUME => Define volumes que devem ser definidos

WORDIR => Define o seu diretório corrente


#### Boas práticas para construção de imagens Docker

##### Nomeando sua imagem Docker

- namespace (a quem pertence a imagem, ex: user docker hub, nome da empresa)
- repositório
- versão da imagem

```dockerfile
bryanlima/api-conversao:v1

bryanlima/api-conversao:latest
```

* Imagens oficiais do Docker não possuem namespace

##### Sempre especifique a tag nas imagens

Má prática
```dockerfile
FROM node
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
```

Boa prática
```dockerfile
FROM node:14.17.5
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
```

##### Aproveitamento das camadas de imagem

Errado
```dockerfile
FROM node:14.17.5
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 8080
CMD ["node", "server.js"]
```

Certo
```dockerfile
FROM node:14.17.5
WORKDIR /app
COPY ./package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
```

##### Docker Registry

- Docker Hub
- Elastic Container Registry
- Azure Container Registry
- Google Container Registry
- Harbor