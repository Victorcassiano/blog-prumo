
Se você está começando a estudar desenvolvimento de software, já deve ter ouvido falar em **Docker**. Mas afinal, o que é Docker, por que ele é tão importante e como começar a usá-lo? Neste guia prático, vamos explicar tudo o que você precisa saber para começar a usar Docker hoje mesmo — mesmo que você nunca tenha visto um container na vida.

----------

## O que é Docker?

Docker é uma **plataforma de virtualização leve** que permite empacotar aplicativos com todas as suas dependências em unidades chamadas **containers**. Diferente de máquinas virtuais tradicionais, os containers compartilham o sistema operacional do host, o que os torna mais rápidos e eficientes.

----------

## Por que usar Docker?

-   **Portabilidade**: seu código funciona do mesmo jeito em qualquer ambiente.
    
-   **Isolamento**: cada container roda de forma isolada, evitando conflitos.
    
-   **Escalabilidade**: ideal para aplicações que precisam crescer rapidamente.
    
-   **Facilidade na entrega**: simplifica a entrega de aplicações para produção.
    

----------

## Conceitos fundamentais

Antes de colocar a mão na massa, é importante entender alguns termos:

|Termo| Descrição |
|--|--|
| **Imagem** | Um modelo para criar containers (como uma foto). |
| **Container**| Uma instância em execução de uma imagem (como uma planta que nasceu da semente).|
| **Dockerfile**| Arquivo com instruções para criar uma imagem.|
| **Docker Hub**| Repositório online de imagens Docker.|
| **Compose**| Ferramenta para orquestrar múltiplos containers.|

----------

## Instalando o docker

### Windows e macOS

1.  Acesse: [Docker](https://www.docker.com/get-started/)
    
2.  Faça a instalação como está na [Documentação](https://docs.docker.com/desktop/).
    
3.  Siga o assistente de instalação.
    
4.  Após instalar, abra o terminal e execute:
    

`docker --version` 

Se aparecer a versão, está tudo certo!

### Linux (Ubuntu/Debian)

    sudo apt update
    sudo apt install docker.io
    sudo systemctl start docker
    sudo systemctl enable docker

----------

## Primeiros passos com Docker

### 1. Rodando seu primeiro container

Execute o comando abaixo para rodar um servidor web simples:

`docker run -d -p 80:80 nginx` 

Acesse: [http://localhost](http://localhost)  
**Parabéns!** Você acabou de rodar seu primeiro container.

----------

### 2. Listando containers

`docker ps` 

----------

### 3. Parando e removendo containers

    docker stop <ID_DO_CONTAINER>
    docker rm <ID_DO_CONTAINER>

----------

## Criando sua própria imagem

Crie um arquivo chamado `Dockerfile` com o seguinte conteúdo:

`FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "index.js"]` 

Depois, execute:

`docker build -t minha-app .
docker run -p 3000:3000 minha-app` 

----------

## Usando Docker Compose

Para aplicações maiores, use o **Docker Compose**. Crie um arquivo `docker-compose.yml`:

    version: '3'
    services:
      web:
        build: .
        ports:
          - "3000:3000"
      db:
        image: postgres
        environment:
          POSTGRES_PASSWORD: exemplo123

Execute com:

`docker-compose up` 

----------

## O que são Dockerfile, docker-compose.yml e .dockerignore?

Agora que você já viu como usar cada um na prática, é importante entender exatamente **para que serve** e **como funciona** cada arquivo.

----------

### Dockerfile — A receita da sua imagem

O **Dockerfile** é um arquivo que contém todas as instruções para construir uma imagem Docker.  
Pense nele como uma _receita de bolo_: cada linha representa um passo.

**Para que serve?**

-   Criar imagens personalizadas
    
-   Definir dependências
    
-   Configurar ambiente e variáveis
    
-   Garantir reprodutibilidade
    

**Como funciona?**

O Docker executa cada instrução de cima para baixo durante o comando:

`docker build -t nome-da-imagem .` 

----------

### docker-compose.yml — Orquestrando múltiplos containers

O `docker-compose.yml` permite iniciar vários containers ao mesmo tempo, cada um com sua configuração específica.

**Para que serve?**

-   Subir API + Banco + Cache com um comando
    
-   Automatizar ambientes de desenvolvimento
    
-   Criar redes e volumes automaticamente
    

**Como funciona?**

`docker-compose up` 

O Compose lê o arquivo inteiro e cria toda a estrutura necessária para rodar sua aplicação.

----------

### .dockerignore — O que NÃO deve ir para dentro do container

O `.dockerignore` funciona como o `.gitignore`, mas para o Docker.

**Para que serve?**

-   Evitar copiar arquivos desnecessários
    
-   Reduzir o tamanho da imagem
    
-   Acelerar o tempo de build
    
-   Impedir que dados sensíveis vazem para dentro do container
    

**Exemplo comum:**

`node_modules
.env .git
logs/
.DS_Store` 

----------

## Dicas de boas práticas

-   Mantenha suas imagens pequenas: use imagens oficiais e minimalistas.
    
-   Use `.dockerignore` para evitar enviar arquivos desnecessários.
    
-   Não execute containers como root.
    
-   Versione suas imagens com tags.
    
-   Monitore o uso de recursos com `docker stats`.
    

----------

Docker não é mais uma ferramenta opcional — é **essencial** para desenvolvedores modernos. Com ele, você garante que seu código rode em qualquer ambiente, facilita a colaboração em equipe e prepara seu projeto para escalar com segurança.

Se você chegou até aqui, já tem o conhecimento necessário para começar a usar Docker nos seus projetos.  
**Que tal colocar isso em prática hoje mesmo?**
