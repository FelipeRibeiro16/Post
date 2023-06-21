# Portfólio Chat Bot assistente e organizador para WhatsApp

Inspirado no post do meu amigo Victor Harry aqui mesmo no Tab News ensinando a criar um bot de sticker no Whatsapp, eu apresento para vocês o meu primeiro projeto para meu portifólio, um chat bot para WhatsApp com o intuito de servir como uma assistente e me ajudar a organizar minhas conversas. Com esse bot, você pode enviar arquivar conversas, transcrever áudios, resumir conversas, criar figurinhas e até mesmo utilizar ChatGPT durante suas conversas. Tudo isso foi feito utilizando Python principalmente à biblioteca Selenium, que me permitiu automatizar o Whatsapp Web.

## 📝 Sumário
- [📝 Sumário](#📝-sumário)
- [📷 Demonstração](#📷-demonstração)
- [🔧 Instalação](#🔧-instalação)
- [🔍 Visão Geral do projeto](#🔍-visão-geral-do-projeto)
- [🚀 Uso](#🚀-uso)
- [💬 Comandos](#💬-comandos)
- [🗣️ Customizando as respostas do ChatGPT](#🗣️-customizando-as-respostas-do-chatgpt)



## 📷 Demonstração
Aqui estão algumas demonstrações do bot em funcionamento:
### Comando para o ChatGPT
![](exemplo_chat.jpg)
### Comando para criar figurinhas
![](exemplo_figurinha.jpg)
### Comando para transcrever áudios
![](exemplo_transcri%C3%A7%C3%A3o.jpg)



## 🔧 Instalação

Para a instalação do projeto basta seguir o README presente no repositorio do projeto no [GitHub](https://github.com/FelipeRibeiro16/Chat-Bot-Whatsapp). Estão presentes tanto a versão inglês como a versão em português do README.

## 🔍 Visão Geral do projeto

O projeto é dividido em 3 partes principais: os modulos do projeto presentes na pasta bot_modules, aonde foram criado métodos para automatizar o WhatsApp, processar as mensagens do chat, utilizar a API do ChatGPT e lidar com as entradas do chat; o arquivo main.py, responsável por executar o bot e utilizar os módulos do projeto; e a pasta data, responsável por armazenar dados, como cookies do Chrome, pasta de downloads do bot, arquivos de configuração e dump de mensagens do bot.

Todas os métodos e funções do projeto possuem descrição de uso, parâmetros e retorno para facilitar o entendimento e utilização.


Aprofundando mais no código do arquivo `main.py` temos a importação dos modulos bem como a biblioteca `dotenv` para carregar as variáveis de ambiente presentes no arquivo .env criado na instalação do projeto, a biblioteca `pathlib` para lidar com os caminhos dos arquivos, a biblioteca `os` para lidar com o sistema operacional e a biblioteca `time` para adicionar delays no código. 
```python
import os
import time
from dotenv import load_dotenv
from pathlib import Path
from bot_modules import whatsapp
from bot_modules import chat as chat_module
from bot_modules import chat_processor, message_processor
from bot_modules import message_summary
from bot_modules import gpt_response
from bot_modules import load_json, save_json, get_closest_match, get_inputs, get_answer, new_input
```

Após a importação dos modulos, temos o carregamento das variáveis de ambiente presentes no arquivo .env e a criação de duas constantes para armazenar o caminho do arquivo `main.py` e a escolha de cachear as respostas do ChatGPT ou não para diminuir o gasto de créditos da API. 
```python
load_dotenv()
CACHING = True if os.getenv("CACHING_RESPONSES") == 'True' else False
WORK_DIRECTORY = os.getcwd()
```

Com isso podemos iniciar o bot, criando uma instância da classe `whatsapp` presente no módulo de mesmo nome e iniciando o bot com o método `start`, após isso o bot irá abrir o WhatsApp Web e aguardar o usuário escanear o código QR exibido no terminal. Com isso é gerado um driver do selenium que será passado para o modulo `chat` para que ele possa utilizar o WhatsApp Web. 

```python
wp = whatsapp()
wp.start()
driver = wp.driver
chat = chat_module(driver)
```

Em seguida o bot já estará em execução e irá aguardar o usuário enviar uma mensagem para o chat principal (Utilizado para evitar que outras pessoas utilizem o bot sem permissão). Na variavél `corresponded` você poderá indicar qual comando o bot irá utilizar para identificar que a mensagem enviada é um comando.
    
```python
corresponded = '/bot'
``` 

Após a definição do chat principal é criado um loop infinito que irá aguardar o usuário enviar um comando
```python
while True:
    chat.rest()
    chat_atual = chat.listen_chats(corresponded)
    message = chat.last_message(chat_atual)
```

A função `rest` irá evitar que o bot fique com um chat aberto que não seja o main chat, para evitar que você perca alguma notificação em outros chats, a função `listen_chats` irá aguardar o usuário enviar um comando e retornar o chat no qual o comando foi enviado, após isso a função `last_message` irá salvar a última mensagem enviada no chat para que ela possa ser utilizada posteriormente.

```python
answers = load_json(Path(f'{WORK_DIRECTORY}/data/bot-config/answers.json'))

input = message_processor(chat_atual['last_message'], corresponded)

inputs = get_inputs(answers['answers'])

match = get_closest_match(input, inputs)
```

O script então irá carregar os comandos salvos no arquivo `answers.json` e irá processar a mensagem enviada pelo usuário para retirar o comando e retornar apenas o texto enviado. Após isso é feito um processamento para identificar qual comando foi enviado e retornar o comando mais próximo, caso o comando enviado não seja reconhecido.

Dado o comando enviado, o script irá executar as funções correspondentes ao comando, como por exemplo o comando `/bot transcreva` que irá utilizar a API whisper da OpenAI através da função `audio_reader`
```python
elif match == 'transcreva':
    transcribe = chat.audio_reader()
    if transcribe:
        chat.reply_message(chat_atual, msg_bot('Bot', transcribe))
        chat.mark_as_replied(chat_atual, message)
    else:
        chat.reply_message(chat_atual, msg_bot(
            'Bot', 'Não foi possível transcrever!'))
        chat.mark_as_replied(chat_atual, message)
```

Você pode mandar uma mensagem como resposta utilizando a função `reply_message` e marcar a mensagem como respondida utilizando a função `mark_as_replied`, caso você não marque a mensagem como respondida o bot irá responder a mesma mensagem infinitamente, então tome cuidado ao implementar novos comandos.

E caso você utilize o comando `\bot sair` o loop infinito será quebrado e o bot irá fechar o navegador e encerrar a execução do script.

## 🚀 Uso
Para utilizar o projeto, execute o arquivo `main.py` com o seguinte comando:
```
python main.py
```

Após a execução, um código QR será exibido no terminal. Escaneie-o utilizando o WhatsApp em seu smartphone para sincronizar o bot.

Em seguida, será exibido no terminal a mensagem "Define main chat..." (Definir chat principal...). Para definir o chat principal, envie uma mensagem para o chat desejado utilizando o comando de configuração no método `listen_set_main_chat(command)`. Tenha cuidado ao escolher o chat principal, pois não será possível alterá-lo sem reiniciar o bot. Certifique-se de escolher um chat seguro no qual outras pessoas não possam enviar mensagens indesejadas. Após definir o chat principal, o bot exibirá a mensagem "'Main chat defined!" (Chat principal definido!) e estará pronto para uso.

## 💬 Comandos
Os comandos do bot devem ser escritos com o prefixo "/bot" (você pode alterar isso no arquivo `main.py`).
Aqui estão os comandos disponíveis:

- `/bot adicionar`: O bot listará os chats que não estão arquivados e você poderá escolher um para adicioná-lo à lista de chats monitorados. Para escolher um chat, envie o número correspondente ao chat listado pelo bot. O bot responderá com "Adicionado com sucesso!" ou "Não foi possível adicionar!" em caso de erro. (disponível apenas no chat principal)

- `/bot sair`: O bot responderá com "Saindo!" e deixará de funcionar. (disponível apenas no chat principal)

- `/bot figurinha`: O bot criará uma figurinha com a última imagem enviada no chat.

- `/bot arquivar`: O bot arquivará todos os chats.

- `/bot arquivar grupo`: O bot arquivará todos os chats de grupos.

- `/bot arquivar chats`: O bot arquivará todos os chats que não são grupos.

- `/bot resumir`: O bot listará as conversas que não estão arquivadas e você poderá escolher uma para resumir as mensagens. Envie o número correspondente à conversa listada pelo bot. O bot responderá com "Extraindo mensagens..." e começará a extrair e processar as mensagens, fornecendo o resumo em seguida. (disponível apenas no chat principal)

- `/bot transcreva`: O bot transcreverá o último áudio enviado no chat.

Se você enviar uma mensagem com o prefixo "/bot" sem um comando específico, o bot utilizará o ChatGPT para responder à sua mensagem.

Você também pode utilizar os modulos presentes para criar novos comandos e personalizar o bot da forma que desejar.

## 🗣️ Customizando as respostas do ChatGPT
Você também pode customizar as respostas do ChatGPT para dar a personalidade que você quiser ao seu bot. Para isso, basta editar o arquivo `\data\bot-config\chat_response_role.json` e editar o campo `content` com a personalidade que você deseja. Você também pode adicionar alguns exemplos de prompt para a personalidade, melhorando a qualidade das respostas do bot.

Eu mesmo utilizei esse exemplo de personalidade para o meu bot:
```json
[
    {
        "role": "system",
        "content": "You are a spiteful assistant, that hates the user and makes jokes of his questions."
    },
    {
        "role": "user",
        "content": "Você pode me ajudar com a minha lição de matemática?"
    },
    {
        "role": "assistant",
        "content": "Ah, matemática? A matéria que você nunca vai entender, assim como suas chances de passar nesta tarefa."
    },
    {
        "role": "user",
        "content": "Como está o clima hoje?"
    },
    {
        "role": "assistant",
        "content": "Por que se dar ao trabalho de perguntar? Não é como se você fosse sair e experimentar o mundo real."
    }
]
```

E o resultado foi esse:
![](exemplo_meu_chat.jpg)
![](exemplo_meu_chat2.jpg)

## 🎉 Conluindo
Bom espero que vocês tenham gostado do projeto, e que ele possa ser útil e divertido para vocês como é para mim. Caso vocês tenham alguma dúvida ou sugestão, por favor, deixem nos comentários, abram uma issue ou me mandem uma mensagem no [LinkedIn](https://www.linkedin.com/in/feliperibeiro16/). E se vocês gostaram do projeto, por favor, deixem uma estrela no repositório, isso ajuda muito.