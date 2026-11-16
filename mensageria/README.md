# Mensageria
Atividade individual 2 | Infraestrutura de Software | ADS - CESAR School - 2026.2

## Perguntas

**1. Por que o consumidor só apaga a mensagem depois de processar? O que aconteceria se apagasse antes?**

Se o consumidor apagasse a pensagem (tirasse da fila) logo ao iniciar o processamento dela, teriamos o risco do consumidor cair e a **mensagem se perder silenciosamente**. Por isso o mecanismo do Visibility Timeout "esconde" a mensagem por um tempo estabelecido esperando a confirmação do consumidor de que a mensagem foi processada e pode ser deletada da fila (sqs.delete_message() no nosso caso).

**2. Com dois consumidores, como as mensagens se dividiram? A ordem foi mantida? O que a documentação do SQS diz sobre ordem nas filas padrão?**

nesse teste as mensagens se dividiram uma pra cada (ver tabela no [5.2](#52-dois-consumidores)).

> Na documentacao do SQS, é especificado que as mensagens podem chegar repetidas ou fora de ordem devido a arquitetura altamente distribuida. Mas tambem especificam que a fila padrao tenta sempre manter a ordem quando possivel. (talvez por isso nos meus runs, sairam certinhas)

https://docs.aws.amazon.com/pt_br/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues.html

**3. A fila padrão garante entrega pelo menos uma vez, e a mesma mensagem pode chegar duas vezes. O que isso exige do consumidor? Como vocês o tornariam idempotente?**

Todo consmidor poderia checar se uma guia ja foi processada consultando um banco que salva o id das guias que ja foram consumidas. Caso seja algo muito critico, poderiamos usar uma tecnica de deduplicacao com ID, como especificado na [documentação do SQS](https://docs.aws.amazon.com/pt_br/AWSSimpleQueueService/latest/SQSDeveloperGuide/using-messagededuplicationid-property.html).

**4. Expliquem o visibility timeout com o que viram na mensagem inválida. Para que serve uma DLQ?**

O visibility deixa uma mensagem invisivel por um tempo predeterminado para evitar processamento redundante. Se o tempo passar e nenhum consumidor enviar o delete_message() a mensagem "volta" pra fila para ser processada.

**5. No Rota Vital, qual a diferença entre o hospital chamar a emissão de guia diretamente (síncrono) e passar por uma fila (assíncrono)? O que se ganha e o que se perde?**

## Passos e evidencias
### 1. Instalar as ferramentas
Instalei AWS CLI v2 e a biblioteca boto3 para python (usando venv). O restante já estava instalado.
![instalacao ok](./evidencias/print-versoes.png)
### 2. Subir a fila
ElasticMQ rodando no docker.
![docker rodando imagem](./evidencias/print-elasticmq-docker-rodando.png)
Configurando as variaveis de ambiente e criando a sqs queue com o aws cli.
![queue criada](./evidencias/print-create-queue.png)
![queue url rodando](./evidencias/print-queue-url.png)
![log no elasticmq](./evidencias/print-log-create-queue.png)
### 3-4. consumidor.py e produtor.py
Criados e colocados na pasta src (pra ficar organizadinho).
![scripts de teste copiados](./evidencias/print-dir-src.png)
### 5.1 Um consumidor
Com um consumidor todas as guias produzidas sao consumidas por ele, na ordem (R-001 a R-020). A mensagem invalida volta pra fila e o `ApproximateReceiveCount` sobe a cada tentativa (1, 2, 3, 4, 5).
![um consumidor](./evidencias/print-consumidor-1.png)
![o produtor envia as mensagens](./evidencias/print-produtor-1.png)
### 5.2 Dois consumidores
Com dois consumidores, nesse teste as mensagens se dividiram uma pra cada:

| PID | requisicoes processadas | total |
|---|---|---|
| 55833 | R-001, 003, 005, 007, 009, 011, 013, 015, 017, 019 | 10 |
| 56283 | R-002, 004, 006, 008, 010, 012, 014, 016, 018, 020 | 10 |

Cada consumidor viu as suas em ordem, mas a ordem geral nao foi mantida (ficou intercalada entre os dois). A divisao certinha foi o que aconteceu aqui, nao uma garantia da fila.

Nao limpei a fila entre o 5.1 e o 5.2, entao a invalida do 5.1 continuou rodando (contador 6 a 11, passando pelos dois consumidores) e o produtor colocou uma invalida nova (contador comecando em 1).
![dois consumidores e um produtor](./evidencias/print-dois-consumidores-5.2.png)
#### um experimento
Com tres consumidores (os dois anteriores mais um novo), o comportamento se mantem: as mensagens se dividiram de tres em tres.

| PID | requisicoes processadas | total |
|---|---|---|
| 56283 | R-001, 004, 007, 010, 013, 016, 019 | 7 |
| 55833 | R-002, 005, 008, 011, 014, 017, 020 | 7 |
| 57442 | R-003, 006, 009, 012, 015, 018 | 6 |

Como rodei o produtor de novo sem limpar a fila, ja tinha mais de uma mensagem invalida girando ao mesmo tempo (por isso o mesmo terminal mostra contadores 41-44 e depois 35-38: sao mensagens diferentes). Sem DLQ, elas nunca saem da fila.
![tres consumidores e um produtor](./evidencias/print-experimento-tres-consumidores.png)
