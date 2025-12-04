## Sumário - Neste artigo você vai ver:

 **1. Introdução**
    
**2.  O que é um Agente de IA**
    
 **3. Entendendo a estrutura**
    
**4.  Tutorial Prático**
  
**5.  Próximos Passos**
    

## 1. Introdução

Você já sentiu que o ChatGPT, apesar de inteligente, fica limitado por estar "preso" na caixa de texto? Ele escreve ótimos e-mails, mas não pode enviá-los. Ele cria fórmulas de planilha, mas não consegue preencher o Excel para você.

Se você quer mudar isso, você precisa criar seu primeiro **Agente de IA no n8n**. Este é um conceito inicial um pouco mais avançado, então se está buscando realmente começar do 0, recomendo a leitura do nosso outro post sobre **[n8n: O que é e como criar sua primeira automação em 10 minutos](https://blog.prumoperformance.com/article/n8n-o-que-e-e-como-criar-sua-primeira-automacao-em-10-minutos)**.

Enquanto a maioria das pessoas usa a IA apenas para gerar texto, o n8n permite que você conecte essa inteligência aos seus aplicativos reais. É a diferença entre ter um consultor que só dá conselhos e um funcionário que executa o trabalho. Neste post, vamos dar o próximo passo na automação e construir um agente capaz de usar ferramentas.

----------

## 2. O que é um Agente de IA

Um Agente de IA é a evolução do fluxo de trabalho tradicional. Pense nele como um "Estagiário Inteligente".

No post anteriormente citado, nós definimos exatamente o que o robô devia fazer (passo A, depois passo B). Num Agente de IA, você dá um objetivo e as ferramentas, e **ele decide ou você escolhe** qual ferramenta e quando usar para resolver o problema. É importante entender que podemos deixar ele decidir ou podemos informar quando utilizar, sendo, normalmente, o melhor cenário o orientarmos.

**Por que ele é diferente?**

1.  **Autonomia:** Ele não segue apenas uma linha reta. Se você pedir "Calcule meu lucro", ele sabe que precisa usar a calculadora. Se pedir "Pesquise sobre...", ele pode até usar o Google ou Wikipedia.
    
2.  **Flexibilidade:** Você não precisa reprogramar o fluxo para cada nova pergunta. O cérebro (LLM) se adapta ao pedido e ao prompt.
    
3.  **Integração:** No n8n, o agente pode acessar seus e-mails, CRM e bancos de dados de forma segura.
    
![enter image description here](https://i.imgur.com/VafRMWZ.png)

**Imagem 1:** _A anatomia de um Agente: O cérebro comanda as ferramentas._
  



## 3. Entendendo a estrutura

Antes de montar o agente, precisamos atualizar nosso dicionário com três termos exclusivos dessa nova era:

-   **Model (Modelo):** É o cérebro. É quem "pensa". Geralmente usamos o o4-mini da **OpenAi** ou **Gemini 2.5-flash**, mas você pode e deve testar outros modelos pra ver qual se comporta melhor.
    
	![enter image description here](https://i.imgur.com/z7HaYXu.png)
_imagem ilustrativa do modelo_

-   **Memory (Memória):** É a capacidade do agente lembrar o que você disse na mensagem anterior. Sem isso, ele esquece seu nome assim que você envia a segunda frase. Para testes simples podemos utilizar a simple memory que é do próprio n8n e não exige nada além de um identificador, mas em contextos reais talvez seja melhor usar um banco de dados mais robusto.
![enter image description here](https://i.imgur.com/4wmyRDj.png)
_imagem ilustrativa da memória_
-   **Tools (Ferramentas):** São as habilidades que você dá ao agente. Pode ser uma "Calculadora", uma "Busca no Wikipedia" ou uma "Habilidade de Consulta ou Escrita em uma planilha". Sem ferramentas, o agente é apenas um chatbot comum.
![enter image description here](https://i.imgur.com/Lv4H8UP.png)
    _imagem ilustrativa de ferramentas_

    



## 4. Tutorial Prático:

Agora, vamos criar um Agente simples, mas poderoso: Um assistente que consegue realizar cálculos matemáticos precisos e consultar a Wikipedia (coisas que o ChatGPT sozinho costuma alucinar ou errar).

#### Passo 1: Configurando o Chat Trigger

Ao contrário do Webhook que usamos antes, para falar com um agente em tempo real, usamos o **"On Chat Message"** (ou via Webhook se for conectar no WhatsApp). Para este teste, usaremos a interface de chat do próprio n8n.

![enter image description here](https://i.imgur.com/JQSeBNM.png)
 **Imagem 2:** _On chat Message (Ou When chat message received) permite testar no próprio n8n._ 

#### Passo 2: O Coração (AI Agent Node)

Adicione o nó **"AI Agent"**. Este é um "Super Nó". Perceba que ele tem várias entradas ao redor dele. É aqui que vamos plugar as outras partes.

![enter image description here](https://i.imgur.com/yU7qTdA.png)
**Imagem 3:** _O Agente de IA e suas ferramentas._
     

#### Passo 3: Dando poderes (Conectando Tools)

Agora precisamos conectar as partes como na imagem acima:

1.  No conector **Model**, ligue um nó de algum modelo, aqui utilizei gemini 2.5-flash.
    
2.  No conector **Memory**, ligue um nó "Window Buffer Memory ou Simple Memory".
    
3.  No conector **Tool**, ligue um nó de ferramenta, aqui estou utilizando o do google sheets.
	    

Agora seu agente tem um cérebro, uma memória curta e duas habilidades específicas.

#### Passo 4: Prompt

Ao clicar duas vezes no "Ai Agent" ou Agente de IA, irá se deparar com a tela da configuração do agente, nela você deve adicionar um "System Prompt como no exemplo:

![enter image description here](https://i.imgur.com/3CXGA59.png)
**Imagem 4:** _Configuração do Agente de IA._
     
Outras opção são possíveis, mas a princípio só precisamos mexer nisso, pois outras coisas já estão configuradas por padrão como o "Prompt (User Message)" que é a mensagem que o agente vai receber do nosso "On Chat Message"

No System Message, você pode colocar o que quiser para guiar o agente a se comportar como deseja. No exemplo vou deixar assim:
![enter image description here](https://i.imgur.com/SLjBbwZ.png)
**Imagem 5:** _Prompt do agente._

Perceba que nesse simples prompt, temos muita informação. 
Primeiro: No canto superior direito da imagem da pra notar que o prompt ta no modo "Expression". Isso significa que meu prompt vai aceitar pequenos trechos de código, como o **{{ $now }}**, que só funciona com o "Expression", ele está em verde porque ta sendo reconhecido. Esse "now" é uma função reservada do n8n que diz a data e hora atual.



#### Passo 5: Testando e Ativando

Ao enviar uma mensagem no chat, o Agente consultará as instruções do prompt e executará a ferramenta necessária.

O que se espera é que ele busque na planilha informações de determinada pessoa e ele diga o horário atual da consulta. No exemplo da imagem abaixo pedi pra me dar informações de Artur:

![enter image description here](https://i.imgur.com/Ug2upnW.png)     
**Imagem 6:** _Processo funcionando com Sucesso!_

As informações foram buscadas de uma planilha que criei para o exemplo, e como se pode ver na imagem acima, o agente retornou perfeitamente as informações.

![enter image description here](https://i.imgur.com/1pKVWZd.png)
**Imagem 7:** _Planilha acessada pelo agente_

----------

## 5. Próximos Passos:

Criar um agente que usa o google sheets para buscar uma informação é apenas o começo, do jeito que pedimos para o agente de ia fazer uma consulta no banco de dados (google sheets), podemos fazer diversas outras operações como escrever na planilha, qualificar um lead como nesse post nosso: [Como qualificar um lead](https://blog.prumoperformance.com/article/como-construir-um-ai-agent-que-qualifica-corretamente-um-lead), mas lógica é a mesma: **Pergunta -> Decisão -> Uso de Ferramenta -> Resposta**.

Dominar os Agentes de IA permite que você crie funcionários digitais que atendem clientes e consultam manuais técnicos da sua empresa. Mas para isso, ele precisa ler seus documentos. É aqui que entra o conceito de RAG, que explicamos detalhadamente no nosso artigo **[RAG de maneira fácil: Como Transformar seu Google Docs em um Cérebro de IA](https://blog.prumoperformance.com/article/rag-de-maneira-facil-como-transformar-seu-google-docs-em-um-cerebro-de-ia)**.

### Links Mencionados

-   **[Como qualificar um lead](https://blog.prumoperformance.com/article/como-construir-um-ai-agent-que-qualifica-corretamente-um-lead)**
-  **[n8n: O que é e como criar sua primeira automação em 10 minutos](https://blog.prumoperformance.com/article/n8n-o-que-e-e-como-criar-sua-primeira-automacao-em-10-minutos)**
    
-   **[RAG de maneira fácil: Como Transformar seu Google Docs em um Cérebro de IA](https://blog.prumoperformance.com/article/rag-de-maneira-facil-como-transformar-seu-google-docs-em-um-cerebro-de-ia)**
