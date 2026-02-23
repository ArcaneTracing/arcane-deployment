# Arcane

![Arcane Hero](https://arcanetracing.com/img/landing_hero_illustration.png)

**OpenTelemetry-Native Observability for AI Systems.**

[**arcanetracing.com**](https://arcanetracing.com) · [**Documentation**](https://arcanetracing.com/docs/intro) · [**Get Started Free**](https://arcanetracing.com/docs/intro) · [**Contact**](mailto:contact@arcanetracing.com)

[![PyPI arcane-sdk](https://img.shields.io/pypi/v/arcane-sdk?label=pypi%20arcane-sdk)](https://pypi.org/project/arcane-sdk/) [![npm arcane-sdk](https://img.shields.io/npm/v/arcane-sdk?label=npm%20arcane-sdk)](https://www.npmjs.com/package/arcane-sdk) [![Docker Pulls](https://img.shields.io/docker/pulls/arcanetracing/arcane?label=docker%20pulls)](https://hub.docker.com/u/arcanetracing)

---

Arcane brings GenAI observability and evaluation on top of OpenTelemetry-compatible backends. Use any tracing backend that exports OTEL and get LLM and agent tracing, datasets, annotations, scores, and auditability — **without vendor lock-in**.

This repo provides Docker Compose configurations to run the Arcane stack locally. [**Deployment docs**](https://arcanetracing.com/docs/more/deployment)

## Prerequisites

- Docker and Docker Compose
- Images are pre-built and published to a Docker registry; Compose pulls them automatically.

## Choose a Compose

| Variant | Trace Backend | Message Broker | Compose File |
|---------|---------------|----------------|--------------|
| Base | None | RabbitMQ | `base/docker-compose.base.yml` |
| Tempo | Tempo | RabbitMQ | `tempo/docker-compose.tempo-rabbitmq.yml` |
| Tempo | Tempo | Kafka | `tempo/docker-compose.tempo-kafka.yml` |
| Jaeger | Jaeger | RabbitMQ | `jaeger/docker-compose.jaeger-rabbitmq.yml` |
| Jaeger | Jaeger | Kafka | `jaeger/docker-compose.jaeger-kafka.yml` |
| ClickHouse | ClickHouse | RabbitMQ | `clickhouse/docker-compose.clickhouse-rabbitmq.yml` |
| ClickHouse | ClickHouse | Kafka | `clickhouse/docker-compose.clickhouse-kafka.yml` |

- **Base**: Minimal run (postgres, rabbitmq, frontend, backend, worker). No trace backend.
- **Trace variants**: Add Grafana Tempo, Jaeger, or ClickHouse for trace storage and querying.

## Quick Start

1. Clone this repo (or navigate to `arcane-deployment`).
2. Copy `.env.example` to `.env` and set values for production, or use defaults for local dev:

```bash
cp .env.example .env
# Edit .env as needed
```

3. Load `.env` into your shell, then run the compose of your choice from the project root:

```bash
# Load env vars (required for compose variable substitution)
set -a && source .env && set +a

# Base (minimal)
docker compose -f base/docker-compose.base.yml up -d

# Example: Tempo + RabbitMQ
docker compose -f tempo/docker-compose.tempo-rabbitmq.yml up -d

# Example: ClickHouse + Kafka
docker compose -f clickhouse/docker-compose.clickhouse-kafka.yml up -d
```

## Access URLs

| Service | URL |
|---------|-----|
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:8085 |
| Jaeger UI | http://localhost:16686 (Jaeger variants only) |
| RabbitMQ Management | http://localhost:15672 (RabbitMQ variants) |
| Tempo | http://localhost:3200 (Tempo variants) |
| ClickHouse | http://localhost:8123 (ClickHouse variants) |
| Kafka | localhost:9092 (Kafka variants) |

**Trace datasource URLs (backend → trace backend):** The backend runs in a container. When adding a Tempo/Jaeger/ClickHouse datasource in Arcane, use either:

| Trace Backend | Datasource URL (use in Arcane) |
|---------------|--------------------------------|
| Tempo | `http://tempo:3200` or `http://localhost:3200` |
| Jaeger | `http://jaeger:16686` |
| ClickHouse | `http://clickhouse:8123` |

Tempo compose files add `extra_hosts` so the backend can reach host-exposed ports via `localhost` (e.g. `http://localhost:3200` for Tempo). For Jaeger/ClickHouse, use the service name (e.g. `http://jaeger:16686`).

## Environment Variables

All compose files use environment variables. Set variables in `.env` and load them into your shell before running compose (e.g. `set -a && source .env && set +a`) so variable substitution (e.g. `${DATABASE_URL:-default}`) works.

### Database

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:password@postgres:5432/arcanedb` |
| `DATABASE_SSL` | Enable SSL for DB | `false` |
| `POSTGRES_USER` | Postgres container user | `user` |
| `POSTGRES_PASSWORD` | Postgres container password | `password` |
| `POSTGRES_DB` | Postgres database name | `arcanedb` |

### Message Broker

**RabbitMQ variants:**

| Variable | Description | Default |
|----------|-------------|---------|
| `RABBITMQ_URL` | RabbitMQ connection URL | `amqp://guest:guest@rabbitmq:5672/` |

**Kafka variants:**

| Variable | Description | Default |
|----------|-------------|---------|
| `KAFKA_BROKERS` | Comma-separated broker list | `kafka:9092` |

### Auth & Security (required in production)

| Variable | Description | Default |
|----------|-------------|---------|
| `BETTER_AUTH_SECRET` | Auth token encryption | `dev-secret-change-in-production` |
| `INTERNAL_API_KEY` | JWT signing key | `dev-jwt-secret` |
| `ENCRYPTION_KEY` | 32+ char key for model config encryption | `dev-encryption-key-64-chars-minimum-required-for-production-safe` |
| `API_KEY_SALT` | Salt for API key hashing | `dev-api-key-salt` |

Generate secure values: `openssl rand -base64 32`

### URLs

| Variable | Description | Default |
|----------|-------------|---------|
| `BETTER_AUTH_URL` | Backend URL (auth callbacks) | `http://localhost:8085` or `http://backend:8085` |
| `FRONTEND_URL` | Frontend URL (CORS, redirects) | `http://localhost:3000` or `http://frontend:3000` |

### SMTP (optional, for invite emails)

| Variable | Description | Default |
|----------|-------------|---------|
| `SMTP_HOST` | SMTP server host | (empty) |
| `SMTP_PORT` | SMTP port | `587` |
| `SMTP_SECURE` | Use TLS | `false` |
| `SMTP_USER` | SMTP username | (empty) |
| `SMTP_PASS` | SMTP password | (empty) |
| `MAIL_FROM` | From address | `no-reply@example.com` |

### Optional

| Variable | Description |
|----------|-------------|
| `ARCANE_ENTERPRISE_LICENSE` | Enterprise license key (base64) |

## Directory Layout

```
arcane-deployment/
├── config/          # OTEL and Tempo configs
├── base/            # Base compose (minimal, no trace backend)
├── tempo/            # Tempo + Kafka/RabbitMQ
├── jaeger/           # Jaeger + Kafka/RabbitMQ
├── clickhouse/       # ClickHouse + Kafka/RabbitMQ
├── .env.example      # Example env file
└── README.md
```

## 💭 Support

- **Documentation** — [arcanetracing.com/docs](https://arcanetracing.com/docs/intro)
- **Contact** — [contact@arcanetracing.com](mailto:contact@arcanetracing.com)
- **GitHub** — [github.com/ArcaneTracing](https://github.com/ArcaneTracing)

## Built on Open Standards. Ready for Production.

Get started for free or schedule a demo to see how Arcane can transform your GenAI observability.

[**Start Free Now**](https://arcanetracing.com/docs/intro) · [**Star on GitHub**](https://github.com/ArcaneTracing)
