# projeto-n8n-clima-c-gemini
Flow feito no n8n para receber pedido no Telegram, com nome de cidade, e responder com o clima do dia, consultando a OpenWeather, usando Gemini para personalisar a mensagem e devolvendo a temperatura atual.

Esta versão também inclui o uso de Google Gemini para melhorar a resposta, sendo esse uso opcional (possivel de configurar com uma variável, com fallback determinístico obrigatório).

## Estrutura

Telegram Trigger → Preparar Entrada → OpenWeather → IF de validação

Sucesso → Fallback Deterministico → Usar Gemini?
- false → Enviar Fallback
- true → Google Gemini → Validar Saida Gemini → Enviar Mensagem Gemini

Erro → Formatar Erro → Enviar Erro

## Entrada
`São Paulo,SP,BR`

O workflow também aceita `São Paulo,SP` e acrescenta `BR`.

## OpenWeather

Endpoint:
`https://api.openweathermap.org/data/2.5/weather`

Parâmetros:
- `q = {{ $json.queue }}`
- `units = metric`
- `lang = pt_br`
- `appid = {{ $env.OPENWEATHER_API_KEY }}`


## Credenciais

Nenhuma chave real está embutida no workflow ou no repositório.

OPENWEATHER_API_KEY: configure como variável de ambiente do n8n. O workflow já utiliza {{ $env.OPENWEATHER_API_KEY }} no nó da OpenWeather.
TELEGRAM_BOT_TOKEN: configure no n8n em Credentials → Telegram API e associe essa credencial aos nós do Telegram.

Se utilizar o Gemini opcional, configure também a credencial correspondente diretamente no n8n.

## Gemini opcional

O nó `Google Gemini - Melhorar Mensagem` usa:
- Google Gemini
- Text / Message a Model
- `models/gemini-2.5-flash`
- temperatura `0.1`
- saída solicitada como JSON: `{"message":"...","ok":true}`

Nenhuma credencial Gemini está incluída.
A temperatura baixa permite resposta objetiva.

### Ativação

Por padrão, o Code node `Fallback Deterministico` contém:

`usar_gemini: false`

Isso garante que a avaliação possa executar o fallback sem credenciais Gemini.

Para usar a versão com Gemini:
1. Configure sua credencial no nó Google Gemini.
2. No node `Fallback Deterministico`, altere `usar_gemini:false` para `usar_gemini:true`.
3. Execute novamente.

O node `Validar Saida Gemini` valida a resposta do Gemini e volta à mensagem determinística se a saída não for um JSON válido.

## Respostas

Sucesso:
`🌤️ A temperatura em Belo Horizonte é de 25°C.`

Erro:
`❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).`

## Importar no n8n

1. Abra Workflows.
2. Escolha Import from File.
3. Selecione `workflow-chatbot-telegram.json`.
4. Configure a credencial Telegram.
5. Configure `OPENWEATHER_API_KEY`.
6. Teste inicialmente com Gemini desativado.
7. Para testar IA, configure a credencial Gemini e altere `usar_gemini` para `true`.

## Testes recomendados

- São Paulo,SP,BR
- Belo Horizonte,MG,BR
- Curitiba,PR,BR
- CidadeQueNaoExiste,SP,BR
