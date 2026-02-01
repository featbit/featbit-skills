---
name: featbit-deployment-docker
description: Expert guidance for deploying FeatBit with Docker Compose across three tiers - Standalone (PostgreSQL only), Standard (PostgreSQL/MongoDB + Redis), and Professional (+ ClickHouse + Kafka). Use when user mentions "docker-compose", "deploy with Docker", "standalone vs standard vs pro", works with docker-compose.yml files, or asks about container configuration, environment variables, or production Docker setup.
appliesTo:
  - "**/docker-compose.yml"
  - "**/docker-compose.yaml"
  - "**/Dockerfile"
license: MIT
metadata:
  version: 2.0.0
  category: deployment
---

# FeatBit Docker Compose Deployment

Expert guidance for deploying FeatBit with Docker Compose. This skill provides deployment instructions for all three tiers with links to detailed configuration files.

## Core Concepts

FeatBit offers three deployment architectures optimized for different scales:

| Tier | Database | Cache | Message Queue | Analytics | Best For |
|------|----------|-------|---------------|-----------|----------|
| **Standalone** | PostgreSQL | None | PostgreSQL | PostgreSQL | Development, testing, teams <10 users |
| **Standard** | PostgreSQL/MongoDB | Redis | Redis | MongoDB | Production, teams 10-50 users |
| **Professional** | PostgreSQL/MongoDB | Redis | Kafka | ClickHouse | Enterprise, 50+ users, millions of requests |

**Quick Selection Guide**:
- **Standalone**: Minimal setup, single server, good for dev/test
- **Standard**: Production-ready with caching, good performance
- **Professional**: Maximum scale, advanced analytics, requires DevOps expertise

📚 **Architecture Details**: https://docs.featbit.co/installation/deployment-options

## Deployment Guides

### Standalone Deployment

**Quick Start**:
```bash
# Download official configuration
curl -O https://raw.githubusercontent.com/featbit/featbit/main/docker-compose.yml

# Start services
docker compose up -d

# Access at http://localhost:8081
# Default login: test@featbit.com / 123456
```

**Prerequisites**: Docker 20.10+, Docker Compose 2.0+, 2GB RAM

📄 **Complete Guide**: [references/standalone-configuration.md](references/standalone-configuration.md)
- Full docker-compose.yml with health checks
- Step-by-step setup instructions
- Using managed PostgreSQL
- Resource requirements
- Limitations and when to upgrade

### Standard Deployment

**Two Options Available**:

**Option A: PostgreSQL + Redis**
- Best for teams familiar with PostgreSQL
- Simpler than MongoDB option

**Option B: MongoDB + Redis** 
- Better for document-oriented workloads
- Requires initialization script

```bash
# Download MongoDB configuration
curl -O https://raw.githubusercontent.com/featbit/featbit/main/docker-compose-mongodb.yml
mv docker-compose-mongodb.yml docker-compose.yml

# Get MongoDB init script
mkdir -p infra/mongodb/docker-entrypoint-initdb.d
curl -o infra/mongodb/docker-entrypoint-initdb.d/v0.0.0.js \
  https://raw.githubusercontent.com/featbit/featbit/main/infra/mongodb/docker-entrypoint-initdb.d/v0.0.0.js

# Start services  
docker compose up -d
```

**Prerequisites**: Docker 20.10+, Docker Compose 2.0+, 4GB RAM (8GB recommended)

📄 **Complete Guide**: [references/standard-configuration.md](references/standard-configuration.md)
- Full configurations for both PostgreSQL and MongoDB options
- Production setup with managed services
- Resource requirements
- When to choose Standard vs Professional

### Professional Deployment

**Enterprise-Scale Architecture**:
- Kafka for high-throughput messaging
- ClickHouse for advanced analytics
- Horizontal scalability
- Handles millions of events per day

```bash
# Download Professional configuration
curl -O https://raw.githubusercontent.com/featbit/featbit/main/docker-compose-pro.yml
mv docker-compose-pro.yml docker-compose.yml

# Start infrastructure first
docker compose up -d zookeeper kafka clickhouse-server postgresql redis
sleep 30

# Start application services
docker compose up -d
```

**Prerequisites**: Docker 20.10+, Docker Compose 2.0+, 8GB+ RAM, 4+ CPU cores

⚠️ **Complexity Warning**: Requires significant DevOps expertise

📄 **Complete Guide**: [references/professional-configuration.md](references/professional-configuration.md)
- Full docker-compose.yml with all services
- Infrastructure startup sequence
- Horizontal scaling configuration
- Using managed services (AWS MSK, ClickHouse Cloud)
- Performance tuning
- Monitoring setup

## Reference Guides

### Environment Variables

Complete reference for all configuration options:

📄 **[references/environment-variables.md](references/environment-variables.md)**
- Provider configuration (DbProvider, MqProvider, CacheProvider)
- Database connection strings (PostgreSQL, MongoDB, Redis, Kafka, ClickHouse)
- UI configuration (API_URL, EVALUATION_URL)
- Service-specific variables
- Using .env files for secrets
- Environment-specific configurations

### Production Best Practices

Security, high availability, backup, and monitoring:

📄 **[references/production-best-practices.md](references/production-best-practices.md)**
- Security (passwords, secrets, SSL/TLS, firewalls, network isolation)
- High availability (health checks, resource limits, managed services)
- Backup and recovery (automated backups, volume snapshots, disaster recovery)
- Monitoring (Prometheus, Grafana, cAdvisor, ELK Stack)
- Performance optimization (database tuning, resource allocation)
- Upgrade strategies (rolling updates, blue-green deployments)

### Troubleshooting

Common issues and solutions:

📄 **[references/troubleshooting.md](references/troubleshooting.md)**
- Port conflicts
- UI connection issues  
- Database connection failures
- Service startup problems
- WebSocket connection failures
- Redis and MongoDB issues
- Kafka and ClickHouse troubleshooting
- Performance problems
- Emergency recovery procedures
- Debugging tips

## Quick Commands

### Service Management

```bash
# Start all services
docker compose up -d

# Stop all services
docker compose down

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

## When to Choose Each Tier

### Choose Standalone If:
✅ Development or testing environment  
✅ Team size <10 users
✅ Low traffic (< 100k requests/day)
✅ Quick evaluation of FeatBit
✅ Simple single-server deployment

❌ Not Recommended For:
- Production deployments
- High-traffic applications
- Need for reliability and performance

### Choose Standard If:
✅ Production environment  
✅ Team size 10-50 users
✅ Moderate to high traffic
✅ Need caching for performance
✅ Production-level reliability required
✅ Good balance of complexity vs features

❌ Consider Professional If:
- Team size >50 users
- Very high traffic (millions of requests/day)
- Need advanced analytics

### Choose Professional If:
✅ Enterprise deployment  
✅ Team size 50+ users
✅ Very high traffic (millions of requests/day)
✅ Need advanced analytics and data warehousing
✅ Event streaming requirements
✅ Have dedicated DevOps resources
✅ Budget for infrastructure

❌ Consider Standard If:
- Limited DevOps expertise
- Moderate traffic requirements
- Cost-sensitive

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
- **MongoDB Init**: https://github.com/featbit/featbit/blob/main/infra/mongodb/docker-entrypoint-initdb.d/v0.0.0.js

### Docker Images
- **Docker Hub**: https://hub.docker.com/u/featbit
- Images: featbit-ui, featbit-api-server, featbit-evaluation-server, featbit-data-analytics-server

### Alternative Deployments
- **Kubernetes**: https://github.com/featbit/featbit-charts
- **Azure Container Apps**: https://github.com/featbit/azure-container-apps
- **Terraform (AWS)**: https://github.com/featbit/featbit-terraform-aws

## Related Skills

- **featbit-deployment-kubernetes**: Kubernetes/Helm deployments with auto-scaling
- **featbit-deployment**: Overview of all deployment options and architecture
- **featbit-getting-started**: Initial setup, creating feature flags, SDK integration
- **featbit-opentelemetry**: Observability and monitoring setup
