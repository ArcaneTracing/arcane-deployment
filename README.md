# Arcane Deployment

Docker Compose configurations to run the Arcane stack locally with different trace backends and message brokers.

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

3. Run the compose of your choice:

```bash
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

## Environment Variables

All compose files use environment variables. Set them in `.env` or export before running.

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
| `JWT_SECRET` | JWT signing key | `dev-jwt-secret` |
| `ENCRYPTION_KEY` | 32+ char key for model config encryption | `dev-encryption-key-32-chars!!` |
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
