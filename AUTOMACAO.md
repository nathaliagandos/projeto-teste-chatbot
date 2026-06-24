# Descrição da Automação do Chatbot

Este projeto implementa uma automação no n8n para receber uma mensagem de chat, buscar dados em uma planilha do Google Sheets e responder com um modelo de IA.

## Objetivo

- Receber a pergunta do usuário via trigger de chat
- Buscar a base de conhecimento em uma planilha do Google Sheets
- Enviar os dados e a pergunta ao modelo Gemini para gerar a resposta
- Retornar ao usuário apenas a resposta encontrada ou informar que não foi possível localizar a informação

## Fluxo de Nodes

1. **When chat message received**
   - Tipo: `@n8n/n8n-nodes-langchain.chatTrigger`
   - Função: inicia o fluxo quando uma nova mensagem de chat chega.
   - Nesta etapa, o node captura o texto enviado pelo usuário e cria o gatilho para as próximas ações.

2. **Get row(s) in sheet**
   - Tipo: `n8n-nodes-base.googleSheets`
   - Função: lê uma planilha do Google Sheets que funciona como base de conhecimento.
   - Configuração:
     - `documentId`: ID da planilha (em `1M1PN1_9y562fwETtyqmwVEyKoW8Nrrt737VYtbf_q_c`)
     - `sheetName`: `gid=0` (Página 1 da planilha)
   - Este node recupera as linhas disponíveis na planilha e passa esses dados adiante.

3. **Message a model**
   - Tipo: `@n8n/n8n-nodes-langchain.googleGemini`
   - Função: envia a pergunta do usuário e os dados da planilha ao modelo de IA do Google Gemini.
   - Configuração de prompt usada:
     - Define que o assistente é um atendente virtual
     - Informa que a base de conhecimento deve ser o conteúdo retornado do Google Sheets
     - Inclui a pergunta do usuário obtida em `$('When chat message received').first().json.chatInput`
     - Indica para procurar a informação na base e responder somente com ela
     - Se não encontrar, deve responder `Não encontrei essa informação na base.`

## Conexões do fluxo

- `When chat message received` → `Get row(s) in sheet`
- `Get row(s) in sheet` → `Message a model`

Isso garante que:
- primeiro seja capturada a mensagem do usuário,
- depois sejam lidos os dados da planilha,
- e por fim o modelo receba todo o contexto para gerar a resposta.

## Credenciais utilizadas

- Google Sheets: conta OAuth2 configurada para acessar a planilha
- Google Gemini (PaLM): conta configurada para permitir chamadas ao modelo `gemini-2.0-flash-lite-001`

## Comportamento final

Quando um usuário envia uma mensagem via chat:
- o n8n inicia o fluxo com o gatilho de chat,
- busca a base de conhecimento no Google Sheets,
- envia o contexto e a pergunta para o Gemini,
- e retorna uma resposta baseada na informação encontrada.

Se a base não contiver a informação requerida, o assistente responde explicitamente que não encontrou a resposta.
