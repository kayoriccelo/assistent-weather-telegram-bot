# Assistente de Clima - Telegram Bot

Chatbot para Telegram desenvolvido no **n8n** que consulta a previsão do tempo de qualquer cidade utilizando a API do **OpenWeatherMap** e retorna a temperatura atual diretamente no chat.

---

## Descrição

O workflow recebe uma mensagem do usuário no Telegram contendo o nome de uma cidade, trata o texto (remove acentos, normaliza formatação), consulta a API do OpenWeatherMap e devolve a temperatura ao usuário em graus Celsius. O sistema conta com tratamento de erros diferenciado para cidade não encontrada, erro de autenticação na API e falhas de conexão.

### Fluxo do Workflow

1. **Telegram Trigger** — Recebe a mensagem do usuário.
2. **Normalizar Dados** (Code Python) — Normaliza o texto: remove acentos, converte para minúsculas e separa cidade/estado (ex.: `São Paulo,SP` → `sao paulo`).
3. **Buscar Clima OpenWeather** (HTTP Request) — Faz a requisição para a API do OpenWeatherMap com a cidade tratada.
   - **Sucesso** →
     4. **Tratar Dados** (Code Python) — Extrai e formata os campos relevantes (`codigo`, `cidade`, `temperatura`, `pais`).
     5. **Validar Resposta API** (IF) — Verifica se o código de retorno é `200`.
        - **Sim** → **Telegram** — Envia a temperatura ao usuário.
        - **Não** → **Telegram - Erro Não Encontrada** — Informa que a cidade não foi encontrada.
   - **Erro** →
     4. **Verificar Tipo de Erro** (IF) — Verifica se o erro contém código `401`.
        - **Sim** → **Telegram - Erro Autenticação** — Informa que a chave da API está inválida.
        - **Não** → **Telegram - Erro Conexão** — Informa erro genérico de conexão.

```
Telegram Trigger → Normalizar Dados → Buscar Clima OpenWeather
                                              │
                                 ┌────────────┴────────────┐
                                 │                         │
                              Sucesso                  Erro HTTP
                                 │                         │
                           Tratar Dados            Verificar Tipo de Erro
                                 │                      │            │
                        Validar Resposta API          401?         Outro
                           │            │              │            │
                        cod=200      cod≠200           │            │
                           │            │              │            │
                       Telegram    Telegram         Telegram    Telegram
                       (temp.)     Erro Não         Erro        Erro
                                   Encontrada       Autenticação Conexão
```

---

## Pré-requisitos

- [n8n](https://n8n.io/) instalado (self-hosted ou n8n Cloud)
- Um Bot criado no Telegram via [@BotFather](https://t.me/BotFather)
- Conta no [OpenWeatherMap](https://openweathermap.org/api) para obter a chave de API

---

## Credenciais Necessárias

O workflow utiliza o **sistema de credenciais nativo do n8n**. Após importar o workflow, você precisará configurar duas credenciais:

| Credencial              | Tipo no n8n                | Descrição                                      | Onde obter                                                                 |
|-------------------------|----------------------------|-------------------------------------------------|----------------------------------------------------------------------------|
| `TELEGRAM_BOT_TOKEN`   | Telegram API               | Token de autenticação do bot no Telegram        | [@BotFather](https://t.me/BotFather) no Telegram                          |
| `OPENWEATHER_API_KEY`  | OpenWeatherMap API         | Chave de acesso à API do OpenWeatherMap         | [openweathermap.org/api_keys](https://home.openweathermap.org/api_keys)    |

**Importante:** O arquivo JSON exportado **não contém tokens ou chaves reais**. Todas as credenciais são gerenciadas pelo sistema nativo de credenciais do n8n e precisam ser configuradas individualmente após a importação.

---

## Instruções de Importação

### 1. Importar o Workflow no n8n

1. Abra o painel do n8n no navegador.
2. No menu lateral, clique em **Workflows**.
3. Clique em **Import from File** e selecione o arquivo `workflow-chatbot-telegram.json`.
4. O workflow **"Assistente Clima"** será carregado no editor.

### 2. Configurar a Credencial do Telegram

1. No n8n, vá em **Settings → Credentials → Add Credential**.
2. Pesquise por **Telegram** e selecione **Telegram API**.
3. No campo **Access Token**, cole o seu `TELEGRAM_BOT_TOKEN`.
4. Salve a credencial.
5. Nos nós **Telegram Trigger**, **Telegram**, **Telegram - Erro Não Encontrada**, **Telegram - Erro Autenticação** e **Telegram - Erro Conexão**, selecione a credencial criada.

### 3. Configurar a Credencial do OpenWeatherMap

1. No n8n, vá em **Settings → Credentials → Add Credential**.
2. Pesquise por **OpenWeatherMap** e selecione **OpenWeatherMap API**.
3. No campo **API Key**, cole a sua `OPENWEATHER_API_KEY`.
4. Salve a credencial.
5. No nó **Buscar Clima OpenWeather**, selecione a credencial criada na seção de autenticação.

### 4. Ativar o Workflow

1. No editor, alterne o switch no canto superior direito para **Active**.
2. O webhook do Telegram será registrado automaticamente.
3. Envie uma mensagem para o seu bot com o nome de uma cidade para testar.

---

## Exemplos de Uso

| Mensagem enviada        | Resposta do bot                                                          |
|-------------------------|--------------------------------------------------------------------------|
| `São Paulo`             | 🌤️ A temperatura em São Paulo é de 22°C.                               |
| `Curitiba,PR`           | 🌤️ A temperatura em Curitiba é de 18°C.                                |
| `London`                | 🌤️ A temperatura em London é de 15°C.                                  |
| `cidadeinexistente`     | ❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).  |

> **Nota:** As temperaturas acima são apenas exemplos ilustrativos. Os valores reais dependem das condições climáticas no momento da consulta.

---

## Tratamento de Erros

O workflow possui três caminhos de erro distintos:

| Cenário                          | Mensagem enviada ao usuário                                                                                      |
|----------------------------------|------------------------------------------------------------------------------------------------------------------|
| Cidade não encontrada (cod 404)  | ❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).                                           |
| Chave da API inválida (cod 401)  | Erro de autenticação: A chave da API OpenWeather está inválida ou expirada. Por favor, verifique as credenciais. |
| Erro genérico de conexão         | Erro ao conectar com o serviço de clima. Por favor, tente novamente em alguns instantes.                         |

---

## Segurança das Credenciais

O JSON do workflow **não contém credenciais reais**. Todas as chaves e tokens são gerenciados pelo sistema de credenciais nativo do n8n, que armazena os segredos de forma criptografada fora do arquivo do workflow.

Para verificar que nenhum token real está presente no arquivo exportado, rode:

```bash
grep -iE "(sk-|bot[0-9]{8,}|[a-f0-9]{32})" workflow-chatbot-telegram.json
```

O resultado não deve retornar nenhuma chave ou token real. Nunca faça commit de arquivos com chaves ou tokens expostos.

---

## Tecnologias Utilizadas

- **n8n** — Plataforma de automação de workflows
- **Telegram Bot API** — Integração com o chatbot
- **OpenWeatherMap API** — Dados meteorológicos
- **Python** — Tratamento e normalização do texto da mensagem
