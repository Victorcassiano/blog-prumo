
Se você está buscando uma forma robusta, escalável e profissional de rodar o **n8n** no seu próprio servidor, utilizando **Queue Mode**, este guia vai te mostrar exatamente como fazer isso com **Docker**, **Nginx**, **Postgres** e **Redis**.  
Antes de entrarmos na prática, vamos entender rapidamente cada uma dessas ferramentas e por que elas são importantes.

## O que é o n8n ?

O **n8n** é uma plataforma de automação de workflows open-source que permite conectar APIs, bancos de dados, serviços e ações personalizadas de maneira visual. Ele é uma alternativa poderosa ao Zapier e Make, com a vantagem de poder ser totalmente self-hosted.

Se você ainda não conhece o básico do n8n, recomendo fortemente a leitura deste guia introdutório antes de avançar:

**Leitura recomendada:** [n8n: o que é e como criar sua primeira automação em 10 minutos](https://blog.prumoperformance.com/article/n8n-o-que-e-e-como-criar-sua-primeira-automacao-em-10-minutos)

### **n8n Normal Mode**

No modo padrão, o n8n executa os workflows dentro do próprio processo da aplicação. É simples, mas tem limitações de escala especialmente quando muitos workflows disparam ao mesmo tempo ou quando há operações mais pesadas.

### **n8n Queue Mode**

No **Queue Mode**, a execução dos workflows é distribuída entre múltiplos _workers_. Isso permite:

-   Escalabilidade horizontal
    
-   Execuções paralelas reais
    
-   Menos travamentos por sobrecarga
    
-   Maior estabilidade em ambientes de alta demanda
    

Neste tutorial, vamos utilizar justamente o **Queue Mode**, pois ele é ideal para ambientes profissionais.

## O que é Docker e por que vamos usa-lo aqui

O **Docker** permite empacotar aplicações em containers ambientes isolados, leves e portáveis. Com ele, podemos rodar o n8n, Banco de dados e Nginx de forma organizada e sem conflito entre dependências.

Se você ainda está começando com Docker, recomendo este artigo:

🔗 **Leitura recomendada:** [Docker para iniciantes: do zero ao primeiro container](https://blog.prumoperformance.com/article/docker-para-iniciantes-do-zero-ao-primeiro-container)

No nosso setup, o Docker Compose será responsável por subir e orquestrar todos os serviços necessários:  
n8n, Nginx, Postgres e Redis.


## O que é o Nginx

O **Nginx** é um servidor web extremamente rápido e eficiente, usado como proxy reverso na maioria das arquiteturas modernas.

No nosso projeto, ele será responsável por:

-   Redirecionar o tráfego HTTPS para o n8n
    
-   Trabalhar com certificados SSL (Let’s Encrypt)
    
-   Gerenciar rotas e conexões com estabilidade
    

Para uma explicação mais completa sobre o Nginx:

🔗 **Leitura recomendada:** [O que é Nginx e como funciona](https://kinsta.com/pt/blog/o-que-e-nginx/)


## O que é Postgres e por que usaremos no n8n

O **PostgreSQL** é um banco de dados relacional robusto, estável e amplamente utilizado.  
No n8n, ele será responsável por armazenar:

-   credenciais
    
-   usuários
    
-   workflows
    
-   execuções
    
-   logs estruturados
    

É o banco preferido para ambientes profissionais e setups self-hosted.


## O que é Redis e seu papel no Queue Mode

O **Redis** é um banco de dados em memória, extremamente rápido.  
No n8n Queue Mode, ele é obrigatório, pois atua como:

-   **fila de mensagens** (onde workflows são enfileirados para os workers)
    
-   estrutura de comunicação entre o processo principal e os workers
    

Sem Redis, o Queue Mode não funciona!

## Por onde começamos?

Antes de subir o n8n em **Queue Mode** com Docker, Nginx, Postgres e Redis, precisamos do ambiente onde tudo vai rodar. E isso significa uma coisa: você vai precisar de um **servidor**.

### 1. Escolhendo onde hospedar

Para rodar o n8n de forma self-hosted, você precisa contratar uma **VPS**, **servidor dedicado** ou algum serviço de computação em nuvem. Existem várias opções no mercado, cada uma com preços e capacidades diferentes.  
Alguns exemplos populares:

-   [**Hostinger**](https://www.hostinger.com/br/1)
    
-   [**HostGator**](https://www.hostgator.com.br/)
    
-   [**AWS**](https://aws.amazon.com/pt/)
    
-   [**DigitalOcean**](https://www.digitalocean.com/)
    

Não existe resposta universal: qualquer provedor que permita criar uma VPS com Linux já serve.

### 2. O passo essencial: instalar o Docker

Independente do provedor escolhido, o ponto mais importante é que você terá acesso a uma máquina quase sempre com Linux onde precisará instalar o **Docker**, isso porque toda a nossa arquitetura será orquestrada com **containers**.

Com o Docker você poderá:

-   Subir o n8n rapidamente
    
-   Rodar Postgres, Redis e Nginx sem conflito
    
-   Manter tudo organizado em um único `docker-compose.yml`
    
-   Atualizar e escalar sem complicação
    

Se você ainda não domina Docker, recomendo fortemente aprender o básico antes de continuar.

🔗 **Leitura recomendada:** [Docker para iniciantes – do zero ao primeiro container](https://blog.prumoperformance.com/article/docker-para-iniciantes-do-zero-ao-primeiro-container)

Depois de instalar o Docker na sua VPS, já teremos o ambiente ideal para iniciar a configuração completa do n8n em Queue Mode!


## Mão na massa!

Agora que você já contratou sua VPS e instalou o Docker, é hora de realmente começar a preparar o ambiente para rodar o n8n em Queue Mode.

### 1. Liberando e configurando portas no Linux

Depois de acessar sua máquina via SSH pela primeira vez, precisamos garantir que ela aceite conexões externas nas portas que iremos utilizar:

-   **22** → SSH
    
-   **80** → HTTP
    
-   **443** → HTTPS
    

Dependendo da sua VPS, isso pode ser feito de duas maneiras:

#### **Usando UFW (Ubuntu)**

    sudo ufw allow OpenSSH
    sudo ufw allow 80
    sudo ufw allow 443
    sudo ufw enable

#### **Ou via painel da hospedagem**

(Toda VPS tem um painel próprio onde você pode liberar portas.)

Assim, sua máquina estará apta a receber conexões e rodar o Nginx futuramente.

## 2. Criando o diretório da aplicação

Agora vamos criar uma pasta dedicada para o projeto do n8n. Assim tudo ficará organizado no servidor.

    mkdir n8n-app 
    cd n8n-app 

## 3. Criando o docker-compose.yml

Agora vamos criar o arquivo responsável por subir os containers:

`nano docker-compose.yml` 

Insira o seguinte conteúdo inicial:

    services:  
	    n8n-editor:  
			image:  n8nio/n8n:stable  
			container_name:  n8n-editor  
			mem_limit:  1.5g  
			cpus:  1 
			volumes:  
				- ./n8n-data:/home/node/.n8n  
			ports:  
				- 5678:5678  
			depends_on:  
				- postgres-n8n  
				- redis
			environment:
		      N8N_ALLOW_ORIGIN: '*'
		      N8N_EDITOR_BASE_URL: http://localhost:5678
		      WEBHOOK_URL: http://localhost:5678
			networks:  
				-  n8n  
			restart:  
				always

## Explicando cada parte do docker-compose.yml

Agora vamos destrinchar esse trecho para entender exatamente o que ele faz.

### **services:**

Aqui ficam todos os containers que sua aplicação vai usar. O primeiro que configuramos é o **n8n-editor**, que é a interface visual do n8n.

----------

### **n8n-editor:**

Esse é o container principal do n8n em modo Editor, responsável pelo painel visual.

#### **image: n8nio/n8n:stable**

Define qual imagem será usada.  
`stable` garante sempre a versão mais confiável e testada.

----------

#### **container_name: n8n-editor**

Damos um nome para o container. Isso facilita para reiniciar, debugar ou remover:

`docker restart n8n-editor` 

----------

#### **mem_limit: 1.5g**

Limita o uso de memória RAM do container.  
Isso evita que o n8n consuma toda memória disponível da VPS.

----------

#### **cpus: 1**

Limita o contêiner a **1 núcleo de CPU (100% de um core)**, evitando que ele consuma mais do que isso, ajuda a manter o servidor estável.

----------

#### **volumes:**

`-  ./n8n-data:/home/node/.n8n` 

Garante que seus dados não se percam quando o container for removido.  
Tudo que o n8n salvar ficará persistido na pasta local `./n8n-data`.

----------

#### **ports:**

`-  5678:5678` 

A porta externa 5678 da VPS será direcionada para a porta interna 5678 do n8n.

----------

#### **depends_on:**

    -  postgres-n8n  
    -  redis

Garante que o n8n só suba **depois** que o Postgres e o Redis estiverem iniciados.

-------
#### **environment:**
- São variáveis de ambientes importantes para configuração e funcionamento do n8n
----------

#### **networks:**

`-  n8n` 

Define que este serviço usará a rede interna chamada `n8n`.  
Essa rede será definida no final do arquivo.

----------

#### **restart: always**

Se o servidor reiniciar ou o container falhar, o Docker sobe automaticamente.


## Ativando o Queue Mode do n8n

Agora que configuramos o container principal do **n8n-editor**, vamos ativar o **Queue Mode**, que é justamente o que permite escalabilidade real, múltiplos workers e processamento paralelo de execuções.

A boa notícia?  
Ativar o Queue Mode no n8n é **muito mais fácil do que parece**.

Basta criar **dois novos serviços** no `docker-compose.yml`:

-   Um para os **workers**
    
-   Outro para os **webhooks**
    

Vamos adicionar o seguinte trecho ao `docker-compose.yml`:

----------

      n8n-workers:
        image: n8nio/n8n:stable
        mem_limit: 1.5g
        cpus: 1.5
        volumes:
          - ./n8n-data:/home/node/.n8n
        depends_on:
          - postgres-n8n
          - redis
        networks:
          - n8n
        restart: always
        deploy:
          replicas: 3
        environment:
          N8N_ALLOW_ORIGIN: '*'
          N8N_EDITOR_BASE_URL: http://localhost:5678
          WEBHOOK_URL: http://localhost:5678
        command: worker
    
      n8n-webhooks:
        image: n8nio/n8n:stable
        container_name: n8n-webhooks
        mem_limit: 512m
        cpus: 0.5
        volumes:
          - ./n8n-data:/home/node/.n8n
        ports:
          - 5677:5678
        depends_on:
          - postgres-n8n
          - redis
        networks:
          - n8n
        restart: always
        environment:
          N8N_ALLOW_ORIGIN: '*'
          N8N_EDITOR_BASE_URL: http://localhost:5678
          WEBHOOK_URL: http://localhost:5678
        command: webhook

## Explicando cada novo serviço

### **1. n8n-workers**

Os **workers** são o coração do Queue Mode.

Eles ficam responsáveis por **executar workflows**, processando tudo o que chega na fila criada no Redis.

#### Trechos importantes:

-   **`deploy.replicas: 3`**  
    Isso cria **3 workers rodando ao mesmo tempo** escalabilidade automática via Docker.  
    Mais workers = mais processamento paralelo.
    
-   **`command: worker`**  
    É aqui que ativamos o Queue Mode.  
    Esse comando transforma o container em um _worker executor_, e não no editor UI.
    
-   **`environment`**  
    Aqui ficam variáveis importantes para o funcionamento correto dos workers, como URLs públicas.
    

----------

### **2. n8n-webhooks**

O serviço de **webhooks** é crucial no Queue Mode.

Ele é responsável por receber eventos externos (como chamadas de APIs), e adicionar essas execuções na fila do Redis para que os workers as processem.

#### Trechos importantes:

-   **`command: webhook`**  
    Assim como os workers, esse comando define o papel do container.  
    Ele passa a ser um servidor dedicado a webhooks.
    
-   **`ports: 5677:5678`**  
    Webhooks precisam de uma porta diferente da interface principal.
    
-   **Ambiente (`environment`)**  
    Garante que os webhooks estejam integrados com o domínio público e com a instância principal.
    

## Por que o Queue Mode funciona tão bem?

Ao separar os papéis em três serviços distintos:

-   **Editor** → n8n-editor
    
-   **Execução** → n8n-workers
    
-   **Webhooks** → n8n-webhooks
    

…você consegue:

-   Escalar execução facilmente
    
-   Manter o editor responsivo mesmo sob carga
    
-   Processar webhooks sem travar
    
-   Distribuir workload sem gargalos
    
-   Manter execuções estáveis mesmo em alto volume
    
Essa é a arquitetura usada por empresas que precisam de volume, estabilidade e velocidade.

## Configurando variáveis de ambiente

Agora que criamos os serviços do **n8n-editor**, **n8n-workers** e **n8n-webhooks**, precisamos configurar as variáveis de ambiente em cada serviço do n8n.

    services:
      n8n-editor:
        image: n8nio/n8n:stable
        container_name: n8n-editor
        mem_limit: 1.5g
        cpus: 1
        ports:
          - 5678:5678
        volumes:
          - ./n8n-data:/home/node/.n8n
        depends_on:
          - postgres-n8n
          - redis
        networks:
          - n8n-queue
        restart: always
        environment:
          N8N_ALLOW_ORIGIN: '*'
          N8N_EDITOR_BASE_URL: http://localhost:5678
          WEBHOOK_URL: http://localhost:5678
    
          DB_TYPE: postgresdb
          DB_POSTGRESDB_HOST: postgres-n8n
          DB_POSTGRESDB_PORT: 5432
          DB_POSTGRESDB_DATABASE: n8n
          DB_POSTGRESDB_USER: "[Adicione seu usuario]"
          DB_POSTGRESDB_PASSWORD: "[Adicione sua senha]"
    
          QUEUE_BULL_REDIS_HOST: redis-n8n
          QUEUE_BULL_REDIS_PORT: 6379
          N8N_RUNNERS_ENABLED: true
          N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS: true
          EXECUTIONS_MODE: queue
          GENERIC_TIMEZONE: America/Sao_Paulo
          N8N_COMMUNITY_PACKAGES_ALLOW: true
          N8N_DEFAULT_BINARY_DATA_MODE: filesystem
          N8N_TRUST_PROXY: true
          N8N_API_RATE_LIMIT: false
          N8N_PROXY_HOPS: "1"
          N8N_PUSH_BACKEND: "websocket"
          N8N_PAYLOAD_SIZE_MAX: 35
          OFFLOAD_MANUAL_EXECUTIONS_TO_WORKERS: true
    
      n8n-workers:
        image: n8nio/n8n:stable
        mem_limit: 1.5g
        cpus: 1.5
        depends_on:
          - postgres-n8n
          - redis
        networks:
          - n8n-queue
        restart: always
        deploy:
          replicas: 3
        volumes:
          - ./n8n-data:/home/node/.n8n
        environment:
          N8N_ALLOW_ORIGIN: '*'
          N8N_EDITOR_BASE_URL: http://localhost:5678
          WEBHOOK_URL: http://localhost:5678
    
          DB_TYPE: postgresdb
          DB_POSTGRESDB_HOST: postgres-n8n
          DB_POSTGRESDB_PORT: 5432
          DB_POSTGRESDB_DATABASE: n8n
          DB_POSTGRESDB_USER: "[Adicione seu usuario]"
          DB_POSTGRESDB_PASSWORD: "[Adicione sua senha]"
    
          QUEUE_BULL_REDIS_HOST: redis-n8n
          QUEUE_BULL_REDIS_PORT: 6379
          N8N_RUNNERS_ENABLED: true
          N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS: true
          EXECUTIONS_MODE: queue
          GENERIC_TIMEZONE: America/Sao_Paulo
          N8N_COMMUNITY_PACKAGES_ALLOW: true
          N8N_DEFAULT_BINARY_DATA_MODE: filesystem
          N8N_TRUST_PROXY: true
          N8N_API_RATE_LIMIT: false
          N8N_PROXY_HOPS: "1"
          N8N_PUSH_BACKEND: websocket
        command: worker
    
      n8n-webhooks:
        image: n8nio/n8n:stable
        container_name: n8n-webhooks
        mem_limit: 512m
        cpus: 1
        volumes:
          - ./n8n-data:/home/node/.n8n
        ports:
          - 5677:5678
        depends_on:
          - postgres-n8n
          - redis
        networks:
          - n8n-queue
        restart: always
        environment:
          N8N_ALLOW_ORIGIN: '*'
          N8N_EDITOR_BASE_URL: http://localhost:5678
          WEBHOOK_URL: http://localhost:5678
    
          DB_TYPE: postgresdb
          DB_POSTGRESDB_HOST: postgres-n8n
          DB_POSTGRESDB_PORT: 5432
          DB_POSTGRESDB_DATABASE: n8n
          DB_POSTGRESDB_USER: "[Adicione seu usuario]"
          DB_POSTGRESDB_PASSWORD: "[Adicione sua senha]"
    
          QUEUE_BULL_REDIS_HOST: redis-n8n
          QUEUE_BULL_REDIS_PORT: 6379
          N8N_RUNNERS_ENABLED: true
          N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS: true
          EXECUTIONS_MODE: queue
          GENERIC_TIMEZONE: America/Sao_Paulo
          N8N_COMMUNITY_PACKAGES_ALLOW: true
          N8N_DEFAULT_BINARY_DATA_MODE: filesystem
          N8N_TRUST_PROXY: true
          N8N_API_RATE_LIMIT: false
          N8N_PROXY_HOPS: "1"
          N8N_PUSH_BACKEND: websocket
        command: webhook

## Agora vamos criar dois serviços essenciais: **Postgres** e **Redis**

Depois de configurar o serviço principal do n8n, precisamos adicionar os bancos que darão suporte ao sistema: **Postgres** (banco de dados) e **Redis** (fila de execuções e cache).

      postgres-n8n:
        image: ankane/pgvector
        restart: always
        container_name: postgres-n8n
        mem_limit: 1.5g
        cpus: 0.5
        volumes:
          - ./postgres-n8n:/var/lib/postgresql/data
        ports:
          - 5432:5432
        networks:
          - n8n
        environment:
          POSTGRES_PASSWORD: [Adicione sua senha]
          POSTGRES_DB: [Adicione seu usuario]
          POSTGRES_USER: [Adicione sua usuario]
    
      redis-n8n:
        image: redis:alpine
        restart: always
        container_name: redis-n8n
        mem_limit: 512m
        cpus: 0.5
        volumes:
          - ./redis-n8n:/data
        ports:
          - 6379:6379
        networks:
          - n8n
  
# **Configuração da rede no docker-compose.yml**

Ao final do seu arquivo, basta adicionar:

    networks:
      n8n:
        driver: bridge


Isso cria uma rede personalizada chamada **n8n**, que permite que todos os serviços se comuniquem entre si usando o nome do container — exatamente como usamos nos exemplos (`postgres-n8n`, `redis-n8n`, `n8n-editor`, etc).


Então seu arquivo **docker-compose.yml** deve ficar dessa forma:

    services:
      n8n-editor:
        image: n8nio/n8n:stable
        container_name: n8n-editor
        mem_limit: 1.5g
        cpus: 1
        volumes:
          - ./n8n-data:/home/node/.n8n
        ports:
          - 5678:5678
        depends_on:
          - postgres-n8n
          - redis
        networks:
          - n8n
        restart: always
        environment:
          N8N_ALLOW_ORIGIN: '*'
          N8N_EDITOR_BASE_URL: http://localhost:5678
          WEBHOOK_URL: http://localhost:5678
          DB_TYPE: postgresdb
          DB_POSTGRESDB_HOST: postgres-n8n
          DB_POSTGRESDB_PORT: 5432
          POSTGRES_PASSWORD: [senha]
          POSTGRES_DB: [nome do banco de dados]
          POSTGRES_USER: [usuário]
          QUEUE_BULL_REDIS_HOST: redis-n8n
          QUEUE_BULL_REDIS_PORT: 6379
          N8N_RUNNERS_ENABLED: true
          N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS: true
          EXECUTIONS_MODE: queue
          GENERIC_TIMEZONE: America/Sao_Paulo
          N8N_COMMUNITY_PACKAGES_ALLOW: true
          N8N_PAYLOAD_SIZE_MAX: 35
          N8N_DEFAULT_BINARY_DATA_MODE: filesystem
          OFFLOAD_MANUAL_EXECUTIONS_TO_WORKERS: true
          N8N_TRUST_PROXY: true
          N8N_API_RATE_LIMIT: false
          N8N_PROXY_HOPS: "1"
          N8N_PUSH_BACKEND: "websocket"
    
      n8n-workers:
        image: n8nio/n8n:stable
        mem_limit: 1.5g
        cpus: 1.5
        volumes:
          - ./n8n-data:/home/node/.n8n
        depends_on:
          - postgres-n8n
          - redis
        networks:
          - n8n
        restart: always
        deploy:
          replicas: 3
        environment:
          N8N_ALLOW_ORIGIN: '*'
          N8N_EDITOR_BASE_URL: http://localhost:5677
          WEBHOOK_URL: http://localhost:5677
          DB_TYPE: postgresdb
          DB_POSTGRESDB_HOST: postgres-n8n
          DB_POSTGRESDB_PORT: 5432
          POSTGRES_PASSWORD: [senha]
          POSTGRES_DB: [nome do banco de dados]
          POSTGRES_USER: [usuário]
          QUEUE_BULL_REDIS_HOST: redis-n8n
          QUEUE_BULL_REDIS_PORT: 6379
          EXECUTIONS_MODE: queue
        command: worker
    
      n8n-webhooks:
        image: n8nio/n8n:stable
        container_name: n8n-webhooks
        mem_limit: 512m
        cpus: 1
        volumes:
          - ./n8n-data:/home/node/.n8n
        ports:
          - 5677:5678
        depends_on:
          - postgres-n8n
          - redis
        networks:
          - n8n
        restart: always
        environment:
          N8N_ALLOW_ORIGIN: '*'
          N8N_EDITOR_BASE_URL: http://localhost:5677
          WEBHOOK_URL: http://localhost:5677
          DB_TYPE: postgresdb
          DB_POSTGRESDB_HOST: postgres-n8n
          DB_POSTGRESDB_PORT: 5432
          POSTGRES_PASSWORD: [senha]
          POSTGRES_DB: [nome do banco de dados]
          POSTGRES_USER: [usuário]
          QUEUE_BULL_REDIS_HOST: redis-n8n
          QUEUE_BULL_REDIS_PORT: 6379
          EXECUTIONS_MODE: queue
        command: webhook
    
      postgres-n8n:
        image: ankane/pgvector
        container_name: postgres-n8n
        restart: always
        mem_limit: 1.5g
        cpus: 0.5
        volumes:
          - ./postgres-n8n:/var/lib/postgresql/data
        ports:
          - 5432:5432
        networks:
          - n8n
        environment:
          POSTGRES_PASSWORD: [senha]
          POSTGRES_DB: [nome do banco de dados]
          POSTGRES_USER: [usuário]
    
      redis:
        image: redis:alpine
        container_name: redis-n8n
        restart: always
        mem_limit: 512m
        cpus: 0.5
        volumes:
          - ./redis-n8n:/data
        ports:
          - 6379:6379
        networks:
          - n8n
    
    networks:
      n8n:
        driver: bridge

## Finalizando a configuração do ambiente do n8n

Depois de criar todo o arquivo `docker-compose.yml`, falta apenas **um último passo essencial** para que o n8n funcione corretamente em _Queue Mode_: criar a chave de criptografia usada pelo sistema.

### 1. Inicie seus containers pela primeira vez

No terminal, dentro da pasta do seu projeto, execute:

`docker compose up -d --build` 

Na **primeira execução**, é normal que o n8n apresente **alguns erros ao tentar acessar**:

http://localhost:5677

Isso acontece porque o n8n ainda **não possui uma chave de criptografia configurada**.


### 2. O volume `n8n-data` será criado automaticamente

Assim que o serviço inicia pela primeira vez, o Docker cria o diretório que usamos como volume do n8n:

`n8n-data` 

Esse diretório guarda as configurações internas do n8n — e **é justamente nele que precisamos criar o arquivo de configuração**.

### 📄 3. Crie o arquivo `config` (sem extensão!)

Agora, acesse o diretório:

`cd n8n-data` 

E crie o arquivo:

`nano config` 

Sim, é literalmente um arquivo **sem extensão**. 😄

Dentro dele, coloque o seguinte conteúdo:

    {
            "encryptionKey": "[sua_chave_aqui]"
    }


###  4. Gerando a chave correta

Para gerar a chave de forma rápida e segura, use:

[https://it-tools.tech/token-generator](https://it-tools.tech/token-generator)

📌 Requisitos importantes:

-   Deve ter **pelo menos 64 caracteres**
    
-   Quanto maior, melhor a segurança
    
-   Você pode escolher caracteres aleatórios e seguros pelo gerador

Por fim, rode o comando no docker:

    docker compose down && docker compose up -d --build
E verá tudo funcionando!

## Criando o serviço do Nginx com Docker

Agora que o ambiente principal do n8n já está configurado, precisamos criar um segundo projeto para rodar o **Nginx**, que será responsável por receber as requisições HTTP/HTTPS e encaminhar para o n8n (via proxy).

###  1. Criando o diretório do Nginx

No terminal, volte para o diretório raiz onde está seu projeto principal e crie uma nova pasta:

    mkdir nginx-app
    cd nginx-app

Esse será o local onde configuraremos tudo relacionado ao Nginx.


## 2. Criando o docker-compose.yml do Nginx

Dentro dessa pasta, crie o arquivo:

`nano docker-compose.yml` 

E insira o seguinte conteúdo:

    services:
      nginx:
        image: nginx:latest
        container_name: nginx
        mem_limit: 256m
        cpus: 0.5
        ports:
          - "80:80"
          - "443:443"
        networks:
          - n8n
        volumes:
          - ./nginx/conf.d:/etc/nginx/conf.d
          - ./nginx/html:/usr/share/nginx/html
          - /etc/letsencrypt:/etc/letsencrypt
        restart: always
    
    networks:
      n8n:
        external: true



## Explicando o porquê dessa configuração

-   **image: nginx:latest** → usamos a versão mais recente do Nginx
    
-   **ports** → expõem as portas 80 (HTTP) e 443 (HTTPS) da máquina real
    
-   **volumes**:
    
    -   `./nginx/conf.d` → onde ficarão nossos arquivos de configuração (server blocks)
        
    -   `./nginx/html` → diretório estático do Nginx (quase não vamos usar)
        
    -   `/etc/letsencrypt` → pasta onde o Certbot salva certificados SSL
        
-   **networks: n8n** → conecta esse Nginx à mesma rede onde estão os containers do n8n, permitindo o proxy funcionar entre eles
    

⚠️ Note que a rede `n8n` precisa ser **externa**, porque ela já foi criada no docker-compose do n8n.  
Por isso usamos:

`networks:  n8n:  external:  true` 

Assim o Docker **não tenta recriar** a rede apenas conecta o Nginx nela.

## Configurando o arquivo do Nginx para apontar para o n8n

Agora que o serviço do **nginx** já foi criado dentro do diretório `nginx-app`, precisamos criar a configuração que fará o Nginx servir o seu n8n pela porta 80 (HTTP).

###  Estrutura esperada dentro do `nginx-app`

    nginx-app/
     ├── docker-compose.yml
     └── nginx/
     └── conf.d/
            └── n8n.conf

### Criando o arquivo de configuração

Crie o diretório onde ficam as configs:

`mkdir -p nginx/conf.d` 

Agora crie o arquivo:

`nano nginx/conf.d/n8n.conf` 

E coloque o seguinte conteúdo:

    server {
        listen 80;
        server_name seu-dominio.com.br;
    
        location / {
            proxy_pass http://n8n:5678/;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }


###  Explicando

-   `server_name seu-dominio.com.br;` — aqui você coloca o domínio que comprou.
    
-   `proxy_pass http://n8n:5678/;` — este é o serviço `n8n` da rede Docker (porque você o colocou na mesma rede do nginx).
    
-   Toda requisição chegará no nginx → será redirecionada para o container do n8n.
    


# Configurando HTTPS com Certbot

Após configurar o servidor em HTTP, o próximo passo é habilitar HTTPS usando o **Certbot**, que é a ferramenta oficial para gerar certificados gratuitos Let's Encrypt.

# Instalando HTTPS com Certbot dentro do container

Depois que o domínio estiver apontando, rode:

`docker exec -it nginx bash` 

E dentro do container:

`apt update
apt install certbot python3-certbot-nginx -y` 

Agora gere o certificado:

`certbot --nginx -d seu-dominio.com.br -d www.seu-dominio.com.br` 

O Certbot vai:

✔ Validar domínio  
✔ Configurar HTTPS automaticamente  
✔ Criar redirecionamento HTTP → HTTPS  
✔ Gerar certificados em `/etc/letsencrypt`

Como você montou `/etc/letsencrypt` como volume no docker-compose, ele fica persistente!

# 🔄 Renovação automática

O Certbot já instala um cron que renova tudo sozinho.

Mas você pode testar usando:

`certbot renew --dry-run`

## Configurando o Nginx + domínio para acessar o n8n de fora

Para que seu n8n rode de forma profissional, com domínio próprio e HTTPS (via Nginx), você precisará de um domínio ou seja, registre um domínio, por exemplo `meun8n.com` e apontá-lo para o seu servidor VPS.

### ✅ Comprar domínio e apontar para o seu servidor

1.  Contrate um domínio em um registrador (pode ser qualquer um por exemplo, registradores comuns ou provedores como Hostinger).
    
2.  Dentro do painel de controle onde você comprou o domínio, vá para a gestão de DNS (zona DNS).
    
3.  Crie um registro **A** que associe seu domínio ao **endereço IP público da sua VPS**. Normalmente você configura:
    
    -   Nome: `@` → aponta para o IP da VPS
        
    -   Nome: `www` → aponta para o mesmo IP (opcional, para versão com “www”)
        
4.  Aguarde a propagação do DNS sso pode levar de alguns minutos até 24 horas para que o domínio comece a apontar corretamente.
    

Se quiser mais detalhes sobre esse processo  quando usar registro A, CNAME, nameservers, etc. recomendo esse guia:  
🔗 **Leitura sugerida:** [Como apontar o nome de domínio para a Hostinger](https://www.hostinger.com/pt/tutoriais/como-apontar-o-nome-de-dominio-para-hostinger?utm_source=chatgpt.com)


### Por que isso é necessário?

O Nginx vai receber solicitações HTTP/HTTPS vindas de um nome de domínio (por exemplo `meun8n.com`) e repassar (“proxy pass”) para o container do n8n. Para que esse fluxo funcione, o domínio precisa **“apontar” para o IP do seu VPS**. Sem isso, quem digitar seu domínio não chegará ao servidor vai dar erro de DNS.

___

Se você chegou até aqui, já percebeu que rodar o **n8n em Queue Mode** não é apenas “colocar um container pra rodar” é construir uma arquitetura **escalável**, **resiliente** e **profissional**, exatamente igual ao que empresas sérias usam em produção.

Agora você tem:

✅ Uma API de automação robusta  
✅ Workers paralelos prontos para alto volume  
✅ Banco de dados otimizado (Postgres + pgvector)  
✅ Redis garantindo filas rápidas e seguras  
✅ Nginx servindo tudo com proxy reverso  
✅ Certificado SSL automático com Certbot  
✅ Domínio próprio apontado corretamente  
✅ Um projeto organizado em duas aplicações Docker independentes

Tudo isso significa **autonomia total**, **menor custo** e **zero dependência de terceiros**.  
A partir daqui, você já pode:

🚀 Criar automações corporativas  
🚀 Escalar para centenas de execuções simultâneas  
🚀 Garantir estabilidade mesmo com cargas pesadas  
🚀 Publicar webhooks para qualquer cliente ou aplicativo  
🚀 Gerenciar sua própria infraestrutura como uma empresa profissional

Esse é literalmente o **padrão ouro** de self-hosting do n8n.

Colocando tudo em prática, você acabou de montar uma stack completa, moderna e robusta para rodar o **n8n em ambiente de produção real** com alta performance, segurança, SSL, domínio próprio e uma arquitetura totalmente escalável.

A partir deste setup, você não só tem seu próprio ambiente de automações, mas também **controle absoluto** sobre armazenamento, processamento, privacidade e custo.  
É exatamente assim que grandes empresas operam seus workflows críticos.

Agora o próximo passo está nas suas mãos [**Crie sua primeira automação aqui**](https://blog.prumoperformance.com/article/n8n-o-que-e-e-como-criar-sua-primeira-automacao-em-10-minutos)
