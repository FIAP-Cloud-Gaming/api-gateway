# Gateway API

API Gateway para o FIAP Cloud Gaming, implementado com YARP (Yet Another Reverse Proxy).

## Funcionalidades

- **Reverse Proxy (YARP)**: Roteamento centralizado para os microservicos Identity, Catalog e Payment
- **Autenticacao JWT**: Validacao centralizada de tokens JWT no gateway
- **Rate Limiting**: Fixed window (100 requests/minuto) em todas as rotas
- **Health Checks**: Monitoramento ativo dos backends (30s interval) e endpoint proprio `/health`
- **Observabilidade**: Serilog (Loki) + OpenTelemetry (Jaeger)

## Rotas

| Rota | Destino | Auth | Rate Limit |
|------|---------|------|------------|
| `/api/auth/{**path}` | Identity API | Nao | Sim |
| `/api/identity/{**path}` | Identity API | Nao | Nao |
| `/api/catalog/{**path}` | Catalog API | Sim | Sim |
| `/api/payment/{**path}` | Payment API | Sim | Sim |

## Execucao Local

```bash
# Apenas o Gateway (requer servicos rodando)
cd Gateway
dotnet run --project src/Gateway.API

# Sistema completo via Docker Compose
cd Gateway
docker compose up -d
```

## Portas (Development)

| Servico | Porta |
|---------|-------|
| Gateway API | 5000 |
| Identity API | 5001 |
| Catalog API | 5002 |
| Payment API | 5003 |
| PostgreSQL (Identity) | 5432 |
| PostgreSQL (Catalog) | 5433 |
| PostgreSQL (Payment) | 5434 |
| MongoDB | 27017 |
| Redis | 6379 |
| Elasticsearch | 9200 |
| Kibana | 5601 |
| RabbitMQ | 5672 / 15672 |
| Jaeger UI | 16686 |
| Grafana | 3000 |
| Loki | 3100 |

## Stack

- .NET 10.0
- YARP 2.3.0
- JWT Bearer Authentication
- Serilog + Grafana Loki
- OpenTelemetry + Jaeger
