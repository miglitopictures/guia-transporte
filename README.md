# guia-transporte

Atividade individual 1 (Serverless) · Infraestrutura de Software · ADS · CESAR School · 2026.2

Função serverless que gera a **guia de transporte** de uma requisição do Rota Vital, rodando localmente com o AWS SAM CLI.

**Caminho escolhido:** A (AWS SAM CLI local, sem conta AWS).

## Pré-requisitos

- Linux Mint (ThinkPad T14 Gen 2, x86_64)
- AWS SAM CLI (versão em `evidencias/print-sam-version.png`)
- Docker Engine, com o usuário no grupo `docker`
- Python 3.12 (runtime da função)

## Como rodar

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
- [ ] Passo 4: chamada por HTTP (`sam local start-api` + `curl`), com e sem `--warm-containers LAZY`
- [ ] Passo 5: estouro do timeout (`demorar: true`) e ajuste para 10 s
- [ ] Respostas às 5 perguntas
- [ ] Vídeo de até 3 min

## Registros

### Passo 3: invocação com evento

| Execução | Resultado | Duration | Init Duration |
|---|---|---|---|
| 1ª (após baixar a imagem) | erro `JSONDecodeError` (body em base64) | 132,36 ms | 0,08 ms |
| 2ª (evento corrigido) | 201, `invocacaoNesteAmbiente: 1` | 99,17 ms | 0,05 ms |
| 3ª | 201, `invocacaoNesteAmbiente: 1` | 103,06 ms | 0,06 ms |

Na primeira execução, o SAM baixou e montou a imagem `public.ecr.aws/lambda/python:3.12` antes de invocar.

Evidências: `evidencias/print-timing-invocacoes.png`, `evidencias/print-primeira-invocacao-erro-base64-encoding.png`.

## Problemas encontrados

1. **Docker sem permissão** (`permission denied ... docker.sock`): resolvido com `sudo usermod -aG docker $USER` e `newgrp docker`.
2. **Body do evento em base64:** o `sam local generate-event ... --body` grava o body codificado em base64 (`isBase64Encoded: true`). O handler do enunciado faz `json.loads` direto no body, o que gerou `JSONDecodeError`. Resolvido trocando o `body` do `events/pedido.json` pelo JSON em texto.

## Perguntas

1. Por que a primeira invocação demorou mais que a segunda? O que acontece na fase de Init?
   > _a responder_
2. O que o campo `invocacaoNesteAmbiente` mostrou com e sem `--warm-containers`?
   > _a responder_
3. O que aconteceu com `demorar = true`? Qual o tempo máximo no Lambda?
   > _a responder_
4. Custo de 1 milhão de execuções (128 MB, 100 ms) em us-east-1:
   > _a responder (com a conta)_
5. Onde uma função assim faria sentido no Rota Vital, e onde não faria?
   > _a responder_

## Uso de IA

- **Ferramenta:** Claude (Anthropic)
- **Finalidade:** orientação na instalação do SAM CLI e do Docker no Linux Mint, e diagnóstico dos erros de permissão do Docker e do body em base64.
- **Validação:** _a preencher_
