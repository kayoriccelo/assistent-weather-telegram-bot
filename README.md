# Assistente de Clima - Telegram Bot

Chatbot para Telegram desenvolvido no **n8n** que consulta a previsão do tempo de qualquer cidade utilizando a API do **OpenWeatherMap**, refina a resposta com **Google Gemini** e retorna a temperatura atual diretamente no chat.

---

## Descrição

O workflow recebe uma mensagem do usuário no Telegram contendo o nome de uma cidade, trata o texto (remove acentos, normaliza formatação), consulta a API do OpenWeatherMap e utiliza o Google Gemini para gerar uma resposta natural e polida antes de devolver a temperatura ao usuário em graus Celsius. Caso o Gemini esteja indisponível, uma mensagem de fallback é enviada automaticamente.

### Fluxo do Workflow

1. **Telegram Trigger** — Recebe a mensagem do usuário.
2. **Code in Python** — Normaliza o texto: remove acentos, converte para minúsculas e separa cidade/estado (ex.: `São Paulo,SP` → `sao paulo`).
3. **HTTP Request** — Faz a requisição para a API do OpenWeatherMap com a cidade tratada.
4. **Edit Fields** — Extrai os campos relevantes da resposta (`codigo`, `cidade`, `temperatura`, `pais`).
5. **Validar Resposta API** — Verifica se o código de retorno é `200`.
   - **Sucesso** →
     6. **Message a model (Gemini)** — Refina o texto da resposta com IA para tom natural.
     7. **Verificar Gemini Sucesso** — Checa se o Gemini retornou texto válido.
        - **Sim** → **Telegram** — Envia a resposta polida ao usuário.
        - **Não** → **Fallback Mensagem Sucesso** → **Telegram** — Envia mensagem padrão sem IA.
   - **Erro** →
     6. **Message a model1 (Gemini)** — Refina a mensagem de erro com IA.
     7. **Verificar Gemini Erro** — Checa se o Gemini retornou texto válido.
        - **Sim** → **Telegram Erro** — Envia a mensagem de erro polida.
        - **Não** → **Fallback Mensagem Erro** → **Telegram Erro** — Envia mensagem de erro padrão.

```
Telegram Trigger → Code in Python → HTTP Request → Edit Fields → Validar Resposta API
                                                                       │
                                                          ┌────────────┴────────────┐
                                                          │                         │
                                                       Sucesso                    Erro
                                                          │                         │
                                                   Gemini (polir)           Gemini (polir)
                                                          │                         │
                                                  Verificar Gemini          Verificar Gemini
                                                     │         │               │         │
                                                    OK       Falha            OK       Falha
                                                     │         │               │         │
                                                 Telegram  Fallback       Telegram   Fallback
                                                           → Telegram     Erro       → Telegram Erro
```

---

## Pré-requisitos

- [n8n](https://n8n.io/) instalado (self-hosted ou n8n Cloud)
- Um Bot criado no Telegram via [@BotFather](https://t.me/BotFather)
- Conta no [OpenWeatherMap](https://openweathermap.org/api) para obter a chave de API
- Conta no [Google AI Studio](https://aistudio.google.com/) para obter a chave da API do Gemini

---

## Variáveis de Credenciais

O workflow depende de três credenciais que precisam ser configuradas após a importação:

| Variável               | Descrição                                      | Onde obter                                                                 |
|------------------------|-------------------------------------------------|----------------------------------------------------------------------------|
| `TELEGRAM_BOT_TOKEN`  | Token de autenticação do bot no Telegram        | [@BotFather](https://t.me/BotFather) no Telegram                          |
| `OPENWEATHER_API_KEY`  | Chave de acesso à API do OpenWeatherMap         | [openweathermap.org/api_keys](https://home.openweathermap.org/api_keys)    |
| `GOOGLE_API_KEY`       | Chave de acesso à API do Google Gemini          | [Google AI Studio](https://aistudio.google.com/apikey)                     |

**Importante:** O arquivo JSON exportado não contém tokens ou chaves reais. Os campos de credenciais estão preenchidos com os placeholders `TELEGRAM_BOT_TOKEN`, `OPENWEATHER_API_KEY` e `GOOGLE_API_KEY`, que devem ser substituídos pelos seus valores reais após a importação.

---

## Instruções de Importação

### 1. Importar o Workflow no n8n

1. Abra o painel do n8n no navegador.
2. No menu lateral, clique em **Workflows**.
3. Clique em **Import from File** e selecione o arquivo `workflow-telegram-chatbot.json`.
4. O workflow **"Assistente Clima"** será carregado no editor.

### 2. Configurar a Credencial do Telegram

1. No n8n, vá em **Settings → Credentials → Add Credential**.
2. Pesquise por **Telegram** e selecione **Telegram API**.
3. No campo **Access Token**, cole o seu `TELEGRAM_BOT_TOKEN`.
4. Salve a credencial.
5. Nos nós **Telegram Trigger**, **Telegram** e **Telegram Erro**, selecione a credencial criada.

### 3. Configurar a Chave da API OpenWeather

1. Clique no nó **HTTP Request** dentro do workflow.
2. Na seção **Query Parameters**, localize o parâmetro `appid`.
3. Substitua o valor `OPENWEATHER_API_KEY` pela sua chave real.
4. Salve.

### 4. Configurar a Credencial do Google Gemini

1. No n8n, vá em **Settings → Credentials → Add Credential**.
2. Pesquise por **Google PaLM (Gemini)** e selecione a opção correspondente.
3. No campo **API Key**, cole o seu `GOOGLE_API_KEY`.
4. Salve a credencial.
5. Nos nós **Message a model** e **Message a model1**, selecione a credencial criada.

### 5. Ativar o Workflow

1. No editor, alterne o switch no canto superior direito para **Active**.
2. O webhook do Telegram será registrado automaticamente.
3. Envie uma mensagem para o seu bot com o nome de uma cidade para testar.

---

## Exemplos de Uso

| Mensagem enviada        | Resposta do bot                                           |
|-------------------------|-----------------------------------------------------------|
| `São Paulo`             | 🌤️ A temperatura em São Paulo/BR é 22°C.                |
| `Curitiba,PR`           | 🌤️ A temperatura em Curitiba/BR é 18°C.                 |
| `London`                | 🌤️ A temperatura em London/GB é 15°C.                   |
| `cidadeinexistente`     | ❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP). |

> **Nota:** As respostas acima são exemplos. O Gemini pode reformular o texto para soar mais natural. Caso o Gemini esteja indisponível, as mensagens de fallback são enviadas no formato exato acima.

---

## Segurança das Credenciais

O JSON do workflow **não contém credenciais reais**, apenas placeholders. Para verificar, rode:

```bash
grep -E "(TELEGRAM_BOT_TOKEN|OPENWEATHER_API_KEY|GOOGLE_API_KEY)" workflow-telegram-chatbot.json
```

O resultado deve mostrar apenas os placeholders, sem nenhum token real. Nunca faça commit de arquivos com chaves ou tokens expostos.

---

## Tecnologias Utilizadas

- **n8n** — Plataforma de automação de workflows
- **Telegram Bot API** — Integração com o chatbot
- **OpenWeatherMap API** — Dados meteorológicos
- **Google Gemini** — Refinamento e polimento das mensagens com IA
- **Python** — Tratamento e normalização do texto da mensagem
- **JavaScript** — Mensagens de fallback quando o Gemini está indisponível
