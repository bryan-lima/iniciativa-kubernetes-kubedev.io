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

---


### Aula 2 - Desvendando o Kubernetes


#### Formas de criar um cluster Kubernetes

- Recursos On-Premisse
- Kubernetes as a Service (KaaS)
- Cluster Kubernetes local

##### On-Premisse

Neste cenário, todo o cluster Kubernetes é de responsabilidade da equipe que implementou.

Existem diversas formas de criar o cluster Kubernetes, as principais ferramentas são:

- KubeAdm (criado pela própria Kubernetes)
- RKE (ferramenta da Suse)
- MicroK8s (criado pela Canonical, é uma solução mais enxuta, para dispositivos IoT ou executar localmente)
- Kubespray
- K3S (também enxuta)

##### Kubernetes as a Service (KaaS)

Neste cenário, o Control Plane é gerenciado pelo serviço de nuvem, e o Worker Node a equipe define o setup (quantidade CPU, memória, se e quantidade de GPU, quantidade de Worker Node no cluster).

Os principais serviços em nuvem são:

- AKS - Azure Kubernetes Service (da Microsoft)
- EKS - Elastic Kubernetes Service (da AWS)
- GKE - Google Kubernetes Engineering
- Oracle
- Digital Ocean

##### Local

- Minikube (da própria Kubernetes)
- K3S
- MicroK8s
- Kind (solução baseada em containers)
- K3D (solução baseada em containers)

#### K3D

Baseado no K3S.

##### Instalação

Acessar https://k3d.io/v5.2.2/#installation e seguir instruções.

#### Kubectl

Ferramenta para interagir com o cluster Kubernetes.

Acessar https://kubernetes.io/docs/tasks/tools/ e seguir instruções.

#### Criando o cluster

Criar cluster Kubernete com apenas 1 container
```cmd
k3d cluster create
```

Listar os nodes do cluster Kubernete
```cmd
kubectl get node
```

Criar cluster definindo nome e sem container de balanceamento de nodes
```cmd
k3d cluster create meucluster --no-lb
```

Listar cluster criados
```cmd
k3d cluster list
```

Deletar cluster default
```cmd
k3d cluster delete
```

Deletar cluster usando nome do cluster
```cmd
k3d cluster delete meucluster
```

Criar cluster definindo quantidade de servers (control plane) e agents (worker nodes)
```cmd
k3d cluster create meucluster --servers 3 --agents 3
```

#### POD

Menor objeto do cluster, é nele que o container é executado.

Compartilha mesmo IP e file system.

Arquivo YAML, sempre terá as 4 características:

- apiVersion
- kind
- metadata
- spec

Verificar apiVersion do Pod
```cmd
kubectl api-resources | grep pod
```

Criar arquivo manifesto pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: meupod
spec: 
  containers:
    - name: web
      image: kubedevio/web-page:blue
      ports: 
        - containerPort: 80
```

Criar pod
```cmd
kubectl create -f pod.yaml
```

Listar pods criados
```cmd
kubectl get pods
```

Ver detalhes de um pod
```cmd
kubectl describe pod/meupod
```

Conectar ao pod para ver o que está rodando no pod
```cmd
kubectl port-forward pod/meupod 8080:80
```

Deletar pod
```cmd
kubectl delete pod meupod
```

#### Label e Selectors

Criar ou atualizar pod
```cmd
kubectl apply -f pod.yaml
```

Listar pods que possui determinado label
```cmd
kubectl get pods -l app=web
```

#### ReplicaSet

Verificar apiVersion do ReplicaSet
```cmd
kubectl api-resources | grep ReplicaSet
```

Criar arquivo manifesto replicaset.yaml
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: meureplicaset
spec: 
  selector:
    matchLabels:
      app: web
  template: 
    metadata: 
      labels:
        app: web
    spec: 
      containers:
        - name: web
          image: kubedevio/web-page:blue
          ports: 
            - containerPort: 80
```

Remover pods declarados no manifesto
```cmd
kubectl delete -f pod.yaml
```

Criar pods gerenciados pelo replicaset
```cmd
kubectl apply -f replicaset.yaml
```

Listar replicaset criados
```cmd
kubectl get replicaset
```

Definir número de réplicas via terminal
```cmd
kubectl scale replicaset meureplicaset --replicas 10
```

Alterar template do pod
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: meureplicaset
spec: 
  replicas: 4
  selector:
    matchLabels:
      app: web
  template: 
    metadata: 
      labels:
        app: web
    spec: 
      containers:
        - name: web
          image: kubedevio/web-page:green
          ports: 
            - containerPort: 80
```

Verificar se houve atualização da imagem
```cmd
kubectl apply -f replicaset.yaml

kubectl port-forward pod/meureplicaset-lvclj 8080:80
```
* Nesse caso não houve atualização automática, é necessário adicionar um controlador


#### Deployment


Criar manifesto deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: meudeployment
spec: 
  replicas: 4
  selector:
    matchLabels:
      app: web
  template: 
    metadata: 
      labels:
        app: web
    spec: 
      containers:
        - name: web
          image: kubedevio/web-page:green
          ports: 
            - containerPort: 80
```

Criar ou atualizar deployment
```cmd
kubectl apply -f deployment.yaml
```

Alterar imagem via terminal
```cmd
kubectl set image deployment meudeployment web=kubedevio/web-page:green
```

Histórico de alterações de um deployment
```cmd
kubectl rollout history deployment meudeployment
```

Realizar rollback de um deployment
```cmd
kubectl rollout undo deployment meudeployment
```

#### Services

Tipos de service:

- ClusterIP
- NodePort
- LoadBalancer

Adicionar configuração de service
```yaml
apiVersion: v1
kind: Service
metadata: 
  name: web
spec: 
  selector: 
    app: web
  ports: 
    - protocol: TCP
      port: 80
  type: NodePort
```

Listar services criados
```cmd
kubectl get service
```

Listar todos os pods, services, deployments e replicaset de uma vez
```cmd
kubectl get all
```

Verificar containers em execução
```cmd
docker container ls
```

Inspecionar um dos containers e obter informação IPAddress
```cmd
docker inspect c3e295a8d4d3
```
* IPAddress = 172.21.0.4

Verificar porta associada a service
```cmd
kubectl get services
```
* Porta = 30362

No navegador acessar url
```
172.21.0.4:30362
```
* Em Windows e Mac, não irá funcionar


Apagar cluster
```cmd
k3d cluster delete meucluster
```

Criar cluster com port bindind e assim ser possível acessar aplicação (principalmente em Windows e Mac)
```cmd
k3d cluster create --agents 3 --servers 3 -p "8080:30000@loadbalancer"
```

Definir porta 30000 para service
```yaml
apiVersion: v1
kind: Service
metadata: 
  name: web
spec: 
  selector: 
    app: web
  ports: 
    - protocol: TCP
      port: 80
      nodePort: 30000
  type: NodePort
```

No navegador acessar url
```
localhost:8080
```
* Dessa vez funciona no Windows e Mac


#### Primeira aplicação no Kubernetes

Realizar fork do projeto no GitHub: https://github.com/KubeDev/rotten-potatoes

Adicionar Dockerfile
```dockerfile
FROM python:3.8-slim-buster
WORKDIR /app
COPY requirements.txt .
RUN python -m pip install -r requirements.txt
COPY . /app
EXPOSE 5000
CMD ["gunicorn", "--workers=3", "--bind", "0.0.0.0:5000", "-c", "config.py", "app:app"]
```

Gerar imagem Docker
```docker cli
docker build -t bryanlimadev/rotten-potatoes:v1 .
```

Logar no Docker Hub
```docker cli
docker login
```

Enviar imagem para Docker Hub
```docker cli
docker push bryanlimadev/rotten-potatoes:v1
```

Criar imagem com tag latest
```docker cli
docker tag bryanlimadev/rotten-potatoes:v1 bryanlimadev/rotten-potatoes:latest
```

Enviar também a imagem com tag latest (boa prática)
```docker cli
docker push bryanlimadev/rotten-potatoes:latest
```

Validar no Docker Hub a imagem enviada e as versões: https://hub.docker.com/repository/docker/bryanlimadev/rotten-potatoes


Criar novo cluster Kubernetes
```cmd
k3d cluster create meucluster --agents 3 --servers 3 -p "8080:30000@loadbalancer"
```

Criar manifesto deployment
```yaml
# Deployment do MongoDB

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
spec:
  selector:
    matchLabels: 
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:5.0.5
        ports:
          - containerPort: 27017
        env:
          - name: MONGO_INITDB_ROOT_USERNAME
            value: mongouser
          - name: MONGO_INITDB_ROOT_PASSWORD
            value: mongopwd

---

# Service do MongoDB

apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  selector: 
    app: mongodb
  ports:
  - port: 27017
  type: ClusterIP
```

Aplicar manifesto deployment

```cmd
kubectl apply -f deployment.yaml
```

Adicionar deployment e service da aplicação web Rotten Potatoes
```yaml
# Deployment da aplicação web Rotten Potatoes

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  selector:
    matchLabels: 
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: bryanlimadev/rotten-potatoes:v1
        ports:
        - containerPort: 5000
        env:
        - name: MONGODB_DB
          value: admin
        - name: MONGODB_HOST
          value: mongodb
        - name: MONGODB_PORT
          value: "27017"
        - name: MONGODB_USERNAME
          value: mongouser
        - name: MONGODB_PASSWORD
          value: mongopwd

---

# Service da aplicação web Rotten Potatoes

apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector: 
    app: web
  ports:
  - port: 80
    targetPort: 5000
    nodePort: 30000
  type: NodePort
```

