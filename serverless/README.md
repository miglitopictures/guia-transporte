# Guia de Transporte Serverless
Atividade individual 1 | Infraestrutura de Software | ADS - CESAR School - 2026.2

[← voltar ao índice do repositório](../README.md)

**Miguel Duarte de Barros | mdb@cesar.school**

---

Função serverless que gera a **guia de transporte** de uma requisição do Rota Vital, rodando localmente com o AWS SAM CLI.

**Caminho escolhido:** A (AWS SAM CLI local, sem conta AWS).

**Vídeo:** https://youtu.be/6_lkfpqW0Ps

## Pré-requisitos

- Linux Mint (ThinkPad T14 Gen 2, x86_64)
- AWS SAM CLI (versão em [print-sam-version.png](evidencias/print-sam-version.png))
- Docker Engine, com o usuário no grupo `docker`
- Python 3.12 (runtime da função)

## Como rodar

Todos os comandos a partir da pasta `serverless/`.

```bash
# 1. build
sam build

# 2. gerar o evento de teste (Passo 3)
sam local generate-event apigateway aws-proxy --method POST --path guias \
    --body '{"requisicaoId":"R-001","hospital":"H-Recife","bolsas":["O-NEG-0142"]}' \
    > events/pedido.json
# ver "Problemas encontrados": o body sai em base64 e precisa ser ajustado

# 3. invocar a função com o evento
sam local invoke -e events/pedido.json
```

## Progresso

- [x] Passo 1: ferramentas instaladas (SAM CLI, Docker, Python 3.12)
- [x] Passo 2: projeto criado com `sam init` (Hello World, python3.12, zip) e `app.py` trocado pelo do enunciado
- [X] Passo 2: trocar no `template.yaml` o `Path: /hello` por `/guias` e o `Method: get` por `post`
- [x] Passo 3: invocação com evento (duas execuções registradas)
- [x] Passo 4: chamada por HTTP (`sam local start-api` + `curl`), com e sem `--warm-containers LAZY`
- [x] Passo 5: estouro do timeout (`demorar: true`) e ajuste para 10 s
- [x] Respostas às 5 perguntas
- [x] Vídeo de até 3 min

## Registros

### Passo 3: invocação com evento

| Execução | Resultado | Duration | Init Duration |
|---|---|---|---|
| 1ª (após baixar a imagem) | erro `JSONDecodeError` (body em base64) | 132,36 ms | 0,08 ms |
| 2ª (evento corrigido) | 201, `invocacaoNesteAmbiente: 1` | 99,17 ms | 0,05 ms |
| 3ª | 201, `invocacaoNesteAmbiente: 1` | 103,06 ms | 0,06 ms |

Na primeira execução, o SAM baixou e montou a imagem `public.ecr.aws/lambda/python:3.12` antes de invocar.

**Evidências:**

[Primeira invocação com erro de base64](evidencias/print-primeira-invocacao-erro-base64-encoding.png)

[Tempos das invocações](evidencias/print-timing-invocacoes.png)

### Passo 4: chamada por HTTP

```bash
sam local start-api                          # rodada 1
sam local start-api --warm-containers LAZY   # rodada 2
# em outro terminal, três vezes em cada rodada:
curl -X POST http://127.0.0.1:3000/guias -H "Content-Type: application/json" \
    -d '{"requisicaoId":"R-003","hospital":"H-Caruaru"}'
```

| Modo | Quando o container nasce | 1ª chamada | 2ª | 3ª | `invocacaoNesteAmbiente` |
|---|---|---|---|---|---|
| `sam local invoke` (Passo 3) | um novo a cada execução | ~100 ms | ~100 ms | | 1, 1 |
| `start-api` sem flag | ao subir o servidor ("Initializing the lambda functions containers") | 112,15 ms | 2,10 ms | 2,18 ms | 1, 2, 3 |
| `start-api --warm-containers LAZY` | na 1ª chamada (`Mounting ... SAM_CONTAINER_ID`) | 100,47 ms | 1,96 ms | 1,74 ms | 1, 2, 3 |

- Nas chamadas reaproveitadas, o log mostra `Lambda function 'HelloWorldFunction' is already running`.
- Ao reiniciar o servidor, o container é descartado e o contador volta para 1.

**Evidências:**

[sam local start-api rodando](evidencias/print-rodando-api-sam-start-api.png)

[Chamadas HTTP e contador](evidencias/print-sem-warmup.png)

### Passo 5: limite de tempo

```bash
curl -X POST http://127.0.0.1:3000/guias -H "Content-Type: application/json" \
    -d '{"requisicaoId":"R-003","demorar":"true","hospital":"H-Caruaru"}'
```

| Timeout no template | Resposta do curl | Log do servidor | Duration |
|---|---|---|---|
| 3 s | `{"message":"Internal server error"}` (HTTP 502) | `Function 'HelloWorldFunction' timed out after 3 seconds` | 3000,00 ms |
| 10 s (após `sam build` e reiniciar o servidor) | 201, guia `G-R-003` | sem erro | 5098,22 ms |

Observação: `"demorar":"true"` (string) tem o mesmo efeito de `true`, porque qualquer string não vazia é verdadeira no `if` do Python.

**Evidências:**

[Timeout de 3 s: erro](evidencias/print-timeout-3s-erro.png)

[Timeout de 10 s: sucesso](evidencias/print-timeout-sucesso-10.png)

[curl antes e depois do ajuste do timeout](evidencias/prunt-curl-antes-e-depois-timeout.png)

## Problemas encontrados

1. **Docker sem permissão** (`permission denied ... docker.sock`): resolvido com `sudo usermod -aG docker $USER` e `newgrp docker`.
2. **Body do evento em base64:** o `sam local generate-event ... --body` grava o body codificado em base64 (`isBase64Encoded: true`). O handler do enunciado faz `json.loads` direto no body, o que gerou `JSONDecodeError`. Resolvido trocando o `body` do `events/pedido.json` pelo JSON em texto.
3. **`start-api` reaproveita o container mesmo sem flag:** o enunciado espera que, sem `--warm-containers`, cada chamada suba um container novo. Na versão do SAM usada aqui, o `start-api` cria o container ao iniciar e o reaproveita por padrão. O contraste "sem reaproveitamento" foi observado no `sam local invoke` do Passo 3, em que cada execução cria um container novo e o contador fica em 1.

## Perguntas

1. Por que a primeira invocação demorou mais que a segunda? O que acontece na fase de Init?
   > Na primeira chamada da nossa função acontece um "cold start": a plataforma precisa inicializar o ambiente de execução (no aws sam, subir o container no docker), carregar o codigo na memoria, iniciar o interpretador de python e rodar tudo que fica fora do handler (os `import` e o `contador = 0`). Isso é a fase de init. As invocacoes seguintes reaproveitam o mesmo ambiente e nao precisam passar de novo pelo init, só executam o handler (no `sam local start-api`, mesmo sem o flag). Nos meus testes a primeira chamada levou ~100 ms e as seguintes ~2 ms.
   > fonte: https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html

2. O que o campo `invocacaoNesteAmbiente` mostrou com e sem `--warm-containers`?
   > nos dois casos o contador foi 1, 2, 3. na versao do sam que usei, o `start-api` sem flag ja cria o container quando o servidor sobe e reaproveita ele entre as chamadas; com `--warm-containers LAZY` o container so nasce na primeira chamada, mas depois tambem é reaproveitado. quando reiniciei o servidor o contador voltou pra 1, porque o container foi descartado. o caso "sem reaproveitamento" apareceu no `sam local invoke` do passo 3: cada execucao criou um container novo e o contador ficou sempre em 1.
   > o contador sobe porque o `contador = 0` fica fora do handler e só roda no init, enquanto o `contador += 1` roda a cada chamada. ou seja, a variavel global sobrevive enquanto o ambiente estiver vivo, mas isso é sorte, nao garantia: quem decide se o ambiente é reaproveitado é a plataforma. o ambiente pode morrer (inatividade, deploy, timeout) e, com muitas requisicoes ao mesmo tempo, o lambda cria varias copias da funcao, cada uma com seu proprio contador. entao estado que importa (tipo numero de guias emitidas) tem que ficar fora da funcao, num banco ou cache. variavel global serve pra coisas caras de criar que podem ser recriadas sem problema, tipo uma conexao com o banco.
3. O que aconteceu com `demorar = true`? Qual o tempo máximo no Lambda?
   > Com `demorar = true` a nossa função realiza um sleep de 5s:
   ```python
      if pedido.get("demorar"):
         time.sleep(5) # passa do Timeout de 3 s do template
   ```
   > Que passa do timeout de 3s que configuramos em `template.yaml`:
   ```yaml
      Globals:
         Function:
            Timeout: 3
   ```
   > Isso faz nossa lambda parar antes de finalizar o processamento de nossa funcao handler e retornar a resposta desejada. Resultando num `Internal server error`.

   > quando aumentamos o timeout pra 10s e rodamos `sam build` de novo, a mesma chamada terminou em ~5,1 s com 201. se o trabalho passar de 15 min (maximo segundo [documentacao](https://docs.aws.amazon.com/lambda/latest/dg/configuration-timeout.html)), nao da pra resolver só aumentando o timeout.
   
   > Poderiamos quebrar em partes menores (uma funcao chamando a proxima ao terminar) e orquestrar as etapas (com algo como [Amazon Step Functions](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)). Cada funcao continua curta e stateless, mas o sistema fica mais complexo. Se o trabalho for naturalmente longo e continuo, pode ser mais prudente optar por outro modelo de execucao, tipo um container sempre ligado.

4. Custo de 1 milhão de execuções (128 MB, 100 ms) em us-east-1:
   > valor base: (US$ 0,20 por milhão de requisições e US$ 0,0000166667 por GB-s)
   
   > memoria: 128 MB = 128 / 1024 = 0,125 GB
   > tempo: 100 ms = 0,1 s

   > uma execucao: 0,125 GB × 0,1 s = 0,0125 GB-s
   > um milhao:    0,0125 × 1.000.000 = 12.500 GB-s

   > preco final = 0,20 + (0,0000166667 *12.500) = US$  = 0,20 + 0,20833375

   > **preco final = US$ 0,40833375**

5. Onde uma função assim faria sentido no Rota Vital, e onde não faria?
   > **faria sentido** em tarefas curtas, sem estado e disparadas por evento:
   > - gerar a guia de transporte quando uma requisicao é atendida (exatamente o que essa funcao faz): roda uma vez, devolve o documento e acabou;
   > - mandar notificacao pro hospital quando o status da requisicao muda (aceita, recusada, atendida);
   > - uma verificacao agendada, tipo uma vez por dia, procurando bolsas perto do vencimento ou estoque baixo no hemocentro.
   >
   > nesses casos a funcao fica parada a maior parte do tempo e só roda (e só é cobrada) quando o evento acontece, o que combina com o modelo serverless.
   >
   > **nao faria sentido** na api principal em spring boot. ela atende o painel o tempo todo, mantem conexao com o banco e valida o jwt em toda requisicao. numa funcao serverless, o cold start de uma aplicacao java grande deixaria o usuario esperando, e com trafego constante um servidor sempre ligado sai mais barato e previsivel. tambem nao funcionariam as estruturas que guardamos em memoria no backend, como a `FilaRequisicoes` e a `ListaEstoque`: como vimos na pergunta 2, o ambiente pode morrer ou ter varias copias, entao a fila se perderia ou ficaria diferente em cada copia. esse estado teria que ir pro banco ou pra um servico de fila.

## Uso de IA

- **Ferramenta:** Claude (Anthropic)
- **Finalidade:** orientação, usando pdf da atividade como contexto, na instalação do SAM CLI e do Docker no Linux Mint, e diagnóstico dos erros de permissão do Docker e do body em base64. Assim como escrita da documentacao
- **Validação:** Validacao manual de todos os outputs de documentacao, todos os passos de codigo, comandos de terminal, e verificoes foram feitas manualmente. 
