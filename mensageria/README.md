# Messageria
Atividade individual 2 | Infraestrutura de Software | ADS - CESAR School - 2026.2


## Passos
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
Criados oe colocados na pasta src (pra ficar organizadinho).
![scripts de teste copiados](./evidencias/print-dir-src.png)
### 5.1 Um consumidor
Com um consumidor todas as guias produzidas sao consumidas por ele.
![um consumidor](./evidencias/print-consumidor-1.png)
![o produtor envia as mensagens](./evidencias/print-produtor-1.png)
### 5.2 Dois consumidores
Com dois consumidores, as mensagens sao distribuidas alternadamente entre eles.
![dois consumidores e um produtor](./evidencias/print-dois-consumidores-5.2.png)
#### um experimento
Com tres consumidores, o comportamento se mantem.
![dois consumidores e um produtor](./evidencias/print-experimento-tres-consumidores.png)