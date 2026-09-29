# Bot de Clima no Telegram com n8n

## Descrição

Este projeto implementa um chatbot no Telegram utilizando n8n para consultar a temperatura atual de cidades brasileiras por meio da API Current Weather da OpenWeather.

O usuário pode enviar, por exemplo, `São Paulo,SP,BR`, `Cuiabá,MT` ou `Belo Horizonte`. O workflow normaliza a entrada, consulta a OpenWeather, valida a resposta e retorna uma mensagem curta e amigável com a temperatura atual.

## Tecnologias utilizadas

- n8n
- Telegram Bot API
- OpenWeather API
- Docker Compose (opcional)

## Funcionamento do workflow

1. **Telegram Trigger** recebe mensagens de texto.
2. **Format Input** salva o texto normalizado na variável `queue` e captura o `chat_id`.
3. O texto é convertido para minúsculas, tem acentos removidos e espaços externos eliminados.
4. **OpenWeather API** consulta `https://api.openweathermap.org/data/2.5/weather`.
5. **Verifica Sucesso** valida `cod = 200` e a presença de `name` e `main.temp`.
6. **Prepara Mensagem** arredonda a temperatura e monta a mensagem de sucesso.
7. Em caso de sucesso, o bot envia a temperatura pelo Telegram.
8. Em caso de erro ou cidade inexistente, o bot envia a mensagem de orientação.

## Parâmetros da OpenWeather

| Parâmetro | Valor |
|---|---|
| `q` | variável `queue` formatada; adiciona `,BR` apenas quando necessário |
| `units` | `metric` |
| `lang` | `pt_br` |
| `appid` | `={{ $env.OPENWEATHER_API_KEY }}` |

## Estrutura do projeto

```text
.
├── workflow-chatbot-telegram.json
├── README.md
├── docker-compose.yml
├── .env.example
└── .gitignore
```

> O arquivo `.env` real não deve ser enviado ao GitHub.

## Variáveis e credenciais

Variáveis esperadas:

- `OPENWEATHER_API_KEY`: chave da API da OpenWeather.
- `TELEGRAM_BOT_TOKEN`: token do bot criado no `@BotFather`.

### OpenWeather

O workflow lê a chave pela variável de ambiente:

```text
OPENWEATHER_API_KEY=SUA_CHAVE_OPENWEATHER
```

### Telegram

Crie no n8n uma credencial do tipo **Telegram API** e cole nela o token criado no `@BotFather`.

Depois da importação, associe essa credencial a **todos os nós Telegram** do workflow, incluindo o `Telegram Trigger` e os nós de envio de mensagens.

A variável `TELEGRAM_BOT_TOKEN` também é documentada no `.env.example` e pode ser utilizada na configuração do ambiente Docker, mas nenhum token real é armazenado no workflow exportado.

## Execução com Docker Compose

1. Copie `.env.example` para `.env`.
2. Preencha suas credenciais somente no arquivo local `.env`.
3. Defina `WEBHOOK_URL` com uma URL HTTPS pública que aponte para o seu n8n.
4. Inicie:

```bash
docker compose up -d
```

5. Acesse o n8n em `http://localhost:5678`.

## Como importar no n8n

1. Abra **Workflows** no n8n.
2. Crie um workflow novo.
3. Abra o menu de opções e escolha **Import from File**.
4. Selecione `workflow-chatbot-telegram.json`.
5. Configure a credencial **Telegram API** com o token do BotFather.
6. Associe a credencial a todos os nós Telegram.
7. Confirme que `OPENWEATHER_API_KEY` está disponível no ambiente do n8n.
8. Salve e ative o workflow.

## Exemplos de uso

Envie uma das mensagens abaixo:

```text
São Paulo,SP,BR
Cuiabá,MT
Belo Horizonte
```

Exemplo de resposta:

```text
🌤️ A temperatura em Belo Horizonte é de 25°C.
```

Em caso de erro:

```text
❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).
```

## Casos de teste

Antes de entregar, teste pelo menos:

| Entrada | Resultado esperado |
|---|---|
| `São Paulo,SP,BR` | Temperatura atual de São Paulo |
| `Cuiabá,MT` | Temperatura atual de Cuiabá |
| `Belo Horizonte` | Temperatura atual de Belo Horizonte |
| `CidadeInventadaXYZ` | Mensagem de cidade não encontrada |

## Segurança

- Nenhuma chave real deve ser versionada.
- `.env` está listado no `.gitignore`.
- Use somente `.env.example` no repositório.
- O workflow exportado não contém tokens nem chaves reais.
- Se uma chave tiver sido publicada ou compartilhada acidentalmente, revogue-a e gere outra antes de usar o projeto.
