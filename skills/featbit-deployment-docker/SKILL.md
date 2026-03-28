---
name: featbit-deployment-docker
description: Expert guidance for deploying FeatBit with Docker Compose across three tiers - Standalone (PostgreSQL only), Standard (PostgreSQL/MongoDB + Redis), and Professional (+ ClickHouse + Kafka). Use when user mentions "docker-compose", "deploy with Docker", "standalone vs standard vs pro", works with docker-compose.yml files, or asks about container configuration, environment variables, or production Docker setup. Do not use for Kubernetes, Helm, AWS ECS/EKS, or cloud-provider-specific deployments.
license: MIT
metadata:
  author: FeatBit
  version: 2.0.0
  category: deployment
---

# FeatBit Docker Compose Deployment

Deploy FeatBit with Docker Compose. Covers all three tiers with links to detailed configuration references.

## Execution Procedure

```
1. DETERMINE tier
   - Ask user about expected concurrent connections, API call volume, and event volume
   - Low/moderate traffic, simple setup → Standalone
   - High connections, need caching, moderate events → Standard
   - High connections AND high event volume → Professional
   - If Standard: ask PostgreSQL vs MongoDB preference

2. CLONE repo
   git clone https://github.com/featbit/featbit.git && cd featbit

3. RUN docker compose
   - Standalone:    docker compose up -d
   - Standard PG:   docker compose -f docker-compose-standard.yml up -d
   - Standard Mongo: docker compose -f docker-compose-mongodb.yml up -d
   - Professional:  docker compose -f docker-compose-pro.yml up -d

4. VERIFY access
   - Open http://localhost:8081
   - Login: test@featbit.com / 123456
   - Confirm all services healthy: docker compose ps

5. CONFIGURE (as needed)
   - Read references/environment-variables.md for env var customization
   - Read references/<tier>-configuration.md for tier-specific tuning
   - Read references/troubleshooting-*.md if services fail health checks
```

## Core Concepts

Choose from three deployment architectures optimized for different scales:

| Tier | Database | Cache | Message Queue | Analytics | Best For |
|------|----------|-------|---------------|-----------|----------|
| **Standalone** | PostgreSQL | None | PostgreSQL | PostgreSQL | Low to moderate concurrent connections, moderate API calls, limited event volume |
| **Standard** | PostgreSQL/MongoDB | Redis | Redis | MongoDB | Moderate to high concurrent connections & API calls, moderate event volume |
| **Professional** | PostgreSQL/MongoDB | Redis | Kafka | ClickHouse | Moderate to high concurrent connections & API calls, high event volume |

**Quick Selection Guide**:
- **Standalone**: Minimal setup, single server, handles low to moderate concurrent WebSocket connections
- **Standard**: Production-ready with caching, handles high concurrent connections with moderate event volume
- **Professional**: Maximum scale for high connections AND high event volume, requires DevOps expertise

**Note**: Traffic includes both concurrent WebSocket connections from frontend clients and API calls to clients - these have different scales.

**Architecture Details**: https://docs.featbit.co/installation/deployment-options

## Deployment Guides

Clone the FeatBit repository first — it contains required init scripts:

```bash
git clone https://github.com/featbit/featbit.git
cd featbit
```

**Access all tiers at**:
- **URL**: http://localhost:8081
- **Default Login**: test@featbit.com / 123456

### Standalone Deployment

**Quick Start**:
```bash
# Clone repository (if not already done)
git clone https://github.com/featbit/featbit.git
cd featbit

# Start services using the included docker-compose.yml
docker compose up -d
```

**Prerequisites**: Docker 20.10+, Docker Compose 2.0+, 2GB RAM

**Complete Guide**: [references/standalone-configuration.md](references/standalone-configuration.md)
- Full docker-compose.yml with health checks
- Step-by-step setup instructions
- Using managed PostgreSQL
- Resource requirements
- Limitations and when to upgrade

### Standard Deployment

Pick one of two database backends:

**Option A: PostgreSQL + Redis**
- Use if the team already runs PostgreSQL
- Simpler than MongoDB option

```bash
# Clone repository (if not already done)
git clone https://github.com/featbit/featbit.git
cd featbit

# Start services with PostgreSQL Standard configuration
docker compose -f docker-compose-standard.yml up -d
```

**Option B: MongoDB + Redis**
- Prefer for document-oriented workloads
- Includes initialization script

```bash
# Clone repository (if not already done)
git clone https://github.com/featbit/featbit.git
cd featbit

# Start services with MongoDB configuration
docker compose -f docker-compose-mongodb.yml up -d
```

**Prerequisites**: Docker 20.10+, Docker Compose 2.0+, 4GB RAM (8GB recommended)

**Complete Guide**: [references/standard-configuration.md](references/standard-configuration.md)
- Full configurations for both PostgreSQL and MongoDB options
- Production setup with managed services
- Resource requirements
- When to choose Standard vs Professional

### Professional Deployment

Deploy the full enterprise stack:
- Kafka for high-throughput messaging
- ClickHouse for advanced analytics via the Data Analytics Server
- Horizontal scalability
- Handles millions of events per day

```bash
# Clone repository (if not already done)
git clone https://github.com/featbit/featbit.git
cd featbit

# Start all services (Docker Compose handles startup order via depends_on)
docker compose -f docker-compose-pro.yml up -d
```

**Prerequisites**: Docker 20.10+, Docker Compose 2.0+, 8GB+ RAM, 4+ CPU cores

**Complexity Warning**: Ensure dedicated DevOps expertise before deploying

**Complete Guide**: [references/professional-configuration.md](references/professional-configuration.md)
- Full docker-compose.yml with all services
- Infrastructure startup sequence
- Horizontal scaling configuration
- Using managed services (AWS MSK, ClickHouse Cloud)
- Performance tuning
- Monitoring setup

## Reference Guides

### Environment Variables

Consult these references for all configuration options:

**[references/environment-variables.md](references/environment-variables.md)**
- Provider configuration (DbProvider, MqProvider, CacheProvider)
- Database connection strings (PostgreSQL, MongoDB, Redis, Kafka, ClickHouse)
- UI configuration (API_URL, EVALUATION_URL)
- Service-specific variables
- Using .env files for secrets
- Environment-specific configurations

**[references/environment-variables-otel.md](references/environment-variables-otel.md)**
- OpenTelemetry configuration variables
- Integration with Seq, Jaeger, Prometheus, Grafana, Datadog
- Testing and troubleshooting OTEL setup

### Troubleshooting

Diagnose issues by category:

**[references/troubleshooting-setup.md](references/troubleshooting-setup.md)**
- Login failures and database initialization
- Port conflicts
- UI connection issues
- Database connection failures
- Service startup problems

**[references/troubleshooting-infrastructure.md](references/troubleshooting-infrastructure.md)**
- WebSocket connection failures
- Redis and MongoDB issues
- Kafka and ClickHouse troubleshooting
- Container, disk space, and performance problems
- Data loss after restart

**[references/troubleshooting-recovery.md](references/troubleshooting-recovery.md)**
- Emergency recovery procedures
- Debugging tips
- Getting help

## Quick Commands

### Service Management

```bash
# Start all services (Standalone)
docker compose up -d

# Start all services (Standard PostgreSQL)
docker compose -f docker-compose-standard.yml up -d

# Start all services (Standard MongoDB)
docker compose -f docker-compose-mongodb.yml up -d

# Start all services (Professional)
docker compose -f docker-compose-pro.yml up -d

# Stop all services (use same -f flag as used for starting)
docker compose down
# or for non-default configs:
docker compose -f docker-compose-standard.yml down

# View status
docker compose ps

# View logs
docker compose logs -f

# View specific service logs
docker compose logs -f api-server

# Restart service
docker compose restart api-server

# Scale service (Professional tier)
docker compose up -d --scale evaluation-server=3
```

### Maintenance

```bash
# Update to latest images
docker compose pull
docker compose up -d

# Clean up
docker system prune -a --volumes

# Backup PostgreSQL
docker compose exec postgresql pg_dump -U postgres featbit > backup.sql

# Backup MongoDB
docker compose exec mongodb mongodump --out /backup --db featbit
```

## Official Resources

### Documentation
- **Installation Guide**: https://docs.featbit.co/installation/docker-compose
- **Deployment Options**: https://docs.featbit.co/installation/deployment-options
- **Infrastructure Components**: https://docs.featbit.co/tech-stack/infrastructure-components
- **FAQ**: https://docs.featbit.co/installation/faq

### Source Code & Configurations
- **Main Repository**: https://github.com/featbit/featbit
- **Standalone**: https://github.com/featbit/featbit/blob/main/docker-compose.yml
- **Standard (MongoDB)**: https://github.com/featbit/featbit/blob/main/docker-compose-mongodb.yml
- **Professional**: https://github.com/featbit/featbit/blob/main/docker-compose-pro.yml
- **PostgreSQL Init**: https://github.com/featbit/featbit/tree/main/infra/postgresql/docker-entrypoint-initdb.d
- **MongoDB Init**: https://github.com/featbit/featbit/blob/main/infra/mongodb/docker-entrypoint-initdb.d

### Docker Images
- **Docker Hub**: https://hub.docker.com/u/featbit
- Images: featbit-ui, featbit-api-server, featbit-evaluation-server, featbit-data-analytics-server

### Alternative Deployments
- **Kubernetes**: https://github.com/featbit/featbit-charts
- **Azure Container Apps**: https://github.com/featbit/featbit-aspire
- **Terraform (AWS)**: https://github.com/featbit/featbit-terraform-aws

