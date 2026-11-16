# Guia de Transporte · Rota Vital
Atividades individuais | Infraestrutura de Software | ADS - CESAR School - 2026.2

**Miguel Duarte de Barros | mdb@cesar.school**

---

O Rota Vital é o projeto integrado do semestre: um sistema para gerenciar e rastrear hemocomponentes na rede pública de saúde. As atividades individuais desta disciplina usam o mesmo exemplo, a **emissão da guia de transporte** de cada requisição de um hospital, cada uma com uma forma diferente de infraestrutura.

## Atividades

| # | Pasta | Tema | Como roda | Vídeo |
|---|---|---|---|---|
| 1 | [`serverless/`](serverless/) | função serverless que gera a guia (chamada síncrona, via HTTP) | AWS SAM CLI local + Docker | [youtu.be/6_lkfpqW0Ps](https://youtu.be/6_lkfpqW0Ps) |
| 2 | [`mensageria/`](mensageria/) | fila compatível com SQS, produtor e consumidores (assíncrono) | ElasticMQ local em Docker + Python/boto3 | [youtu.be/Gg44Q7OpnN0](https://youtu.be/Gg44Q7OpnN0) |

Cada pasta tem o próprio README com pré-requisitos, como rodar, registros com evidências, respostas às perguntas e uso de IA.

## Estrutura

```
guia-transporte/
├── serverless/     # atividade 1: AWS SAM (template.yaml, hello_world/, events/, evidencias/)
├── mensageria/     # atividade 2: ElasticMQ (src/, evidencias/, requirements.txt)
└── .gitignore
```
