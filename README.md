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

### Pre-requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [GitHub CLI](https://cli.github.com/) (`gh`)
- `GITHUB_TOKEN` configurado no shell (ver abaixo)

### Configuracao do GitHub Token

Os microsservicos dependem de pacotes NuGet privados do GitHub Packages. O token e necessario para o Docker build.

```bash
# 1. Autenticar no GitHub CLI
gh auth login
gh auth refresh --scopes read:packages

# 2. Adicionar ao perfil do shell:

# PowerShell (notepad $PROFILE):
$env:GITHUB_TOKEN = $(gh auth token)

# Bash (~/.bashrc):
export GITHUB_TOKEN=$(gh auth token)

# 3. Reabrir o terminal e verificar:
# PowerShell: echo $env:GITHUB_TOKEN
# Bash: echo $GITHUB_TOKEN
```

> O token e passado ao Docker via BuildKit secrets e nao fica salvo nas imagens.

### Subir o sistema completo

```bash
# Primeiro build (compila as imagens)
cd Gateway
docker compose up --build -d

# Execucoes seguintes
docker compose up -d

# Se erro de container duplicado
docker compose down && docker compose up -d
```

### Apenas o Gateway (requer servicos rodando)

```bash
cd Gateway
dotnet run --project src/Gateway.API
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
