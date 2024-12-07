# Desafio-dio-Pipeline-de-Desploy
Criando um pipeline de Deploy de uma Aplicação utlizando Gitlab, dcoker e kubernetes

Criando um Pipeline de Deploy de uma Aplicação com GitLab, Docker e Kubernetes
Introdução
Com a evolução da infraestrutura de TI e a adoção de metodologias ágeis, a automação do deploy de aplicações tornou-se uma necessidade crucial para equipes de desenvolvimento. Um dos métodos mais eficientes para gerenciar essa automação é a utilização de CI/CD (Integração Contínua e Entrega Contínua). Nesse contexto, GitLab, Docker e Kubernetes são ferramentas poderosas que, quando combinadas, possibilitam a criação de pipelines robustos e escaláveis.

Neste artigo, vamos aprender como criar um pipeline de deploy utilizando GitLab para a integração contínua, Docker para criar containers e Kubernetes para orquestrar a aplicação em um ambiente escalável.

Pré-requisitos
Antes de começarmos, certifique-se de que você tenha os seguintes requisitos:

GitLab: Uma conta no GitLab com um repositório onde a aplicação estará armazenada.
Docker: Instalar o Docker para construir e armazenar imagens de container.
Kubernetes: Ter um cluster Kubernetes configurado (pode ser um cluster local ou em um provedor de nuvem como Google Kubernetes Engine - GKE, AWS EKS ou Azure AKS).
kubectl: Ferramenta de linha de comando para interagir com o Kubernetes.
Docker Registry: Uma solução de repositório de imagens de containers, como Docker Hub ou GitLab Container Registry.
Passo 1: Configurando o Repositório GitLab
Primeiro, você precisa de um repositório GitLab onde o código da aplicação será armazenado. Certifique-se de que o repositório tenha um arquivo Dockerfile e o código-fonte da aplicação que você deseja implantar.

Crie um repositório no GitLab (se você ainda não tiver um).
Faça o push do seu código-fonte para o repositório.
O Dockerfile deve conter as instruções para construir a imagem do container, por exemplo:

Dockerfile
Copiar código
# Use a imagem oficial do Node.js como base
FROM node:14

# Defina o diretório de trabalho
WORKDIR /app

# Copie o package.json e instale as dependências
COPY package.json /app/
RUN npm install

# Copie o restante do código-fonte
COPY . /app/

# Exponha a porta em que a aplicação rodará
EXPOSE 3000

# Comando para iniciar a aplicação
CMD ["npm", "start"]
Passo 2: Criando o Pipeline no GitLab
GitLab oferece uma ferramenta chamada GitLab CI/CD para automatizar o processo de build, testes e deploy. Para isso, você precisa criar um arquivo .gitlab-ci.yml no seu repositório.

Exemplo de .gitlab-ci.yml
yaml
Copiar código
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_REGISTRY: "docker.io"
  DOCKER_IMAGE: "$CI_REGISTRY/$CI_PROJECT_PATH"

# Fase de Build
build:
  stage: build
  script:
    - docker build -t $DOCKER_IMAGE:$CI_COMMIT_REF_NAME .
    - docker push $DOCKER_IMAGE:$CI_COMMIT_REF_NAME
  only:
    - master

# Fase de Teste
test:
  stage: test
  script:
    - echo "Rodando os testes..."
    - npm test
  only:
    - master

# Fase de Deploy
deploy:
  stage: deploy
  script:
    - kubectl apply -f kubernetes/deployment.yaml
  only:
    - master
  when: manual
stages: Define as etapas do pipeline: build, test e deploy.
variables: Define as variáveis do Docker registry e a imagem que será criada.
build: Na fase de build, o Docker constrói a imagem da aplicação e a envia para o repositório de imagens.
test: Executa os testes definidos para garantir que a aplicação funcione corretamente.
deploy: Realiza o deploy da aplicação no Kubernetes usando o arquivo de configuração YAML (que será discutido no próximo passo).
Passo 3: Configurando o Kubernetes
Agora, precisamos criar o arquivo de configuração Kubernetes para o deploy da nossa aplicação.

Exemplo de deployment.yaml
yaml
Copiar código
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minha-aplicacao
spec:
  replicas: 3
  selector:
    matchLabels:
      app: minha-aplicacao
  template:
    metadata:
      labels:
        app: minha-aplicacao
    spec:
      containers:
      - name: minha-aplicacao
        image: docker.io/usuario/minha-aplicacao:latest
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: minha-aplicacao-service
spec:
  selector:
    app: minha-aplicacao
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
Deployment: Define o número de réplicas da aplicação (neste caso, 3), o nome da aplicação e a imagem do Docker que será usada.
Service: Exponha a aplicação para acesso externo. Aqui, configuramos um serviço do tipo LoadBalancer, que irá alocar um IP externo para acesso à aplicação.
Passo 4: Configurando o GitLab Runner
Para que o pipeline do GitLab funcione corretamente, você precisa ter um GitLab Runner configurado. O GitLab Runner é responsável por executar as etapas do pipeline.

No GitLab, vá até Settings > CI / CD > Runners.
Registre um GitLab Runner em uma máquina local ou servidor que tenha o Docker e o kubectl configurados.
Configure o GitLab Runner para executar os jobs definidos no seu arquivo .gitlab-ci.yml.
Passo 5: Executando o Pipeline
Com tudo configurado, o pipeline será executado automaticamente quando você fizer o push das alterações para o repositório. Durante a execução do pipeline, as seguintes etapas ocorrerão:

Build: O Docker cria a imagem da aplicação e a envia para o repositório de imagens.
Test: A aplicação é testada para garantir que está funcionando corretamente.
Deploy: A aplicação é implantada no Kubernetes usando o arquivo de configuração deployment.yaml.
Conclusão
Neste artigo, aprendemos a criar um pipeline de deploy para uma aplicação utilizando GitLab, Docker e Kubernetes. Esse processo automatiza o ciclo de vida da aplicação, desde o commit do código até o deploy em produção, garantindo que a aplicação seja facilmente escalável e gerenciável.

Com a integração dessas ferramentas, é possível criar um fluxo contínuo de integração e entrega, melhorando a eficiência da equipe e garantindo mais estabilidade na aplicação em produção.
