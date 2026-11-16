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
