**Neste artigo você vai ver:**
1.  **Introdução:** O fim das tarefas repetitivas.
   
2.  **O que é o n8n:** Diferenciais, _fair-code_ e hospedagem própria (Self-hosted).
    
3.  **Os Conceitos Básicos e O Vocabulário:** Entendendo Workflows, Nós, Gatilhos e Ações.
    
4.  **Tutorial Prático (Mão na Massa):**
    -   Configurando o Webhook.        
    -   A Ação (Google Sheets)
        
    -   Mapeando os dados (Drag and Drop).
        
    -   Testando e Ativando.
        
5.  **Próximos Passos, do simples ao futuro:** Do básico aos Agentes de IA com RAG.

## 1. Introdução

Você já se pegou fazendo a mesma tarefa repetitiva no computador todos os dias? Copiar dados de um e-mail para uma planilha, avisar a equipe no Slack quando uma venda cai, ou organizar arquivos no Drive?

Se a resposta for sim, você precisa conhecer o **[n8n](https://n8n.io/)**.

Enquanto ferramentas famosas como Zapier ou Make cobram caro à medida que você cresce, o n8n surgiu como uma alternativa poderosa, flexível e, em muitos casos, muito mais barata (ou até gratuita como explicarei melhor a frente). Neste post, vamos desmistificar essa ferramenta e criar juntos sua primeira automação.

----------

## 1. O que é n8n

O n8n é uma ferramenta de automação de fluxo de trabalho baseada em nós. Pense nele como um "Lego digital".

Cada bloco (nó) faz uma coisa específica: um bloco lê um e-mail, outro bloco processa o texto, e um terceiro bloco salva no Google Sheets. Você conecta esses blocos com linhas para definir o caminho que os dados devem seguir.

**Por que ele é diferente?**

1.  **Visual:** Você vê exatamente o que está acontecendo.
    
2.  **Fair-code:** O código é aberto. Você pode usar a versão em nuvem (paga) ou instalar no seu próprio servidor auto-hospedado de graça através de empresas de hospedagem como a **[Hostinger](https://www.hostinger.com/br)**, que é o nosso caso na **[Prumo Performance](https://prumoperformance.com/).**
    
3.  **Poderoso:** Ele permite desde automações simples até a criação de Agentes de Inteligência Artificial complexos.
    
![ _A interface do n8n: conectando blocos para criar soluções._](https://i.imgur.com/wG3rKmj.png)
  
  **Imagem 1:** _A interface do n8n: conectando blocos para criar soluções._


----------

## 3. Os Conceitos Básicos e O Vocabulário

Antes de abrir a ferramenta, você precisa entender três termos para não ficar perdido:

-   **Workflow (Fluxo):** É o mapa **completo** da sua automação. É a "receita do bolo".

- **Webhook (Gatilho):** Um webhook é um mecanismo que permite que um sistema envie informações automaticamente para outro sistema assim que algum evento acontece
    
-   **Nodes (Nós):** São os ingredientes. Existem dois tipos principais:
    
    -   **Trigger (Gatilho):** O que inicia a automação (Na nossa imagem acima representado pelo símbolo do gmail "Recebe Email").
        
    -   **Action (Ação):** O que acontece depois (exemplos da imagem acima: "Trata Email", "Guarda informações do email no google sheets").
        
-   **Connections (Conexões):** As linhas que ligam os nós e transportam os dados de um passo para o outro.
    
----------

## 4. Tutorial Prático (Mão na Massa):

Agora, vamos criar uma automação clássica e muito útil: Levar dados de uma mensagem do whatsapp (que virá de um Webhook) direto para o Google Sheets.

#### Passo 1: Configurando Webhook

Ao criar um novo workflow, o primeiro passo é definir como ele começa. Vamos usar o nó **Webhook**. Ele gera uma URL única; sempre que alguém acessar essa URL (ou enviar dados para ela), sua automação roda.

![](https://i.imgur.com/ZuiOX6n.png)

**Imagem 2:** _O Webhook gera um endereço exclusivo para receber seus dados, na imagem borrei porque estamos usando o servidor auto-hospedado da prumo performance._
     

#### Passo 2: A Ação (Google Sheets)

Agora, adicione o nó do **Google Sheets**. Você precisará conectar sua conta do Google (o n8n facilita isso com uma autenticação segura, mas provavelmente você terá que permitir no google console). Escolha a operação: "Append Row" (Adicionar linha). Isso fará com que o n8n escreva uma nova linha na sua planilha sempre que o gatilho for acionado.

![enter image description here](https://i.imgur.com/2zoOOHn.png)

**Imagem 3:** _O workflow vai ficar assim, no print eu já executei para testar._

#### Passo 3: Mapeando os dados (Drag and Drop)

Essa é a mágica do n8n. Quando você testa o Webhook, ele traz dados. Para passar esses dados para a planilha, você não precisa programar. Basta abrir o nó do Google Sheets e **arrastar** a informação que veio do Webhook para o campo da coluna da planilha.

Exemplo: Arraste o `nome` do Webhook para a coluna `Nome` da planilha, neste exemplo estou pegando as informações do whatsapp através da evolution Api.

![enter image description here](https://i.imgur.com/7FJiK82.png)

**Imagem 4:** _Informações do webhook_

![enter image description here](https://i.imgur.com/nCb1A3K.png)

**Imagem 5:** _Informação recebida do webhook_

#### Passo 4: Testar e Ativar

Clique em "Execute Workflow". Se tudo der certo, você verá os nós ficarem verdes e a informação aparecerá magicamente na sua planilha. Não esqueça de mudar o botão "Inactive" para **"Active"** no topo da tela para deixar seu robô trabalhando 24h por dia.

![enter image description here](https://i.imgur.com/PuL8Rvv.png)

**Imagem 6:** _Sucesso! As informações foram escritas automaticamente no google sheets ao executar a automação._

----------

## 5.  Próximos Passos, do simples ao futuro:

O n8n pode parecer intimidante no começo por causa de tantas opções, mas a lógica é sempre a mesma: **Gatilho -> Processamento -> Ação**.

Dominar o n8n abre portas para criar desde assistentes pessoais simples até sistemas complexos com Inteligência Artificial, mas não que responde qualquer coisa, existem técnicas para controlar e limitar o que seu agente vai responder, como falamos no nosso artigo **[RAG de maneira fácil: Como Transformar seu Google Docs em um Cérebro de IA](https://blog.prumoperformance.com/article/rag-de-maneira-facil-como-transformar-seu-google-docs-em-um-cerebro-de-ia)**. Isso permite criar um Agente de Inteligência Artificial treinado para sua empresa.


### Links Mencionados

- **[n8n](https://n8n.io/)**
- **[Hostinger](https://www.hostinger.com/br)**
- **[Prumo Performance](https://prumoperformance.com/).**
- **[RAG de maneira fácil: Como Transformar seu Google Docs em um Cérebro de IA](https://blog.prumoperformance.com/article/rag-de-maneira-facil-como-transformar-seu-google-docs-em-um-cerebro-de-ia)**
