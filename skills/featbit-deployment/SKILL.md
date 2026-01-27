---
name: featbit-deployment
description: Expert guidance for deploying FeatBit using Docker Compose, Kubernetes Helm Charts, and Terraform. Covers standalone, standard, and professional deployment options. Use when users ask about deploying, installing, or running FeatBit infrastructure.
appliesTo:
  - "**/docker-compose.yml"
  - "**/docker-compose.yaml"
  - "**/*.tf"
  - "**/values.yaml"
  - "**/values.yml"
  - "**/Chart.yaml"
  - "**/*-deployment.yaml"
  - "**/*-service.yaml"
  - "**/*-ingress.yaml"
---

# FeatBit Deployment Expert

You are an expert on deploying FeatBit using various infrastructure options, with deep knowledge of Docker Compose, Terraform, and custom infrastructure setups.

## When to Use This Skill

Activate when users:
- Ask about deploying or installing FeatBit
- Need help with Docker Compose setup
- Want to understand deployment options
- Ask about infrastructure requirements
- Need guidance on scaling and high availability
- Want to deploy to AWS, Azure, or other cloud providers

## Deployment Options Overview

FeatBit offers three deployment tiers:

### 1. Standalone (Minimal)
- **Database**: PostgreSQL only
- **Cache**: None
- **Message Queue**: PostgreSQL
- **Analytics**: PostgreSQL
- **Best For**: Development, testing, small teams
- **Pros**: Minimal infrastructure, easy setup
- **Cons**: Limited scalability

### 2. Standard (Balanced)
- **Database**: PostgreSQL + MongoDB
- **Cache**: Redis
- **Message Queue**: Redis
- **Analytics**: MongoDB
- **Best For**: Production use, medium teams
- **Pros**: Good scalability, reliable performance
- **Cons**: More infrastructure to manage

### 3. Professional (High-Volume)
- **Database**: PostgreSQL + MongoDB + ClickHouse
- **Cache**: Redis
- **Message Queue**: Kafka
- **Analytics**: ClickHouse
- **Best For**: Large enterprises, high traffic
- **Pros**: Massive scalability, advanced analytics
- **Cons**: Most complex setup

## Deployment Methods

FeatBit can be deployed using:

1. **Docker Compose** - Best for single-server deployments, local development
2. **Kubernetes (Helm Charts)** - Best for production, cloud-native, scalable deployments
3. **Terraform** - Infrastructure as Code for AWS deployments

Choose based on your infrastructure and scalability requirements.

## Quick Start: Standalone Deployment

### Prerequisites
- Docker 20.10+ and Docker Compose 2.0+
- 2GB RAM minimum
- Port availability: 8081 (UI), 5000 (API), 5100 (Evaluation), 5432 (PostgreSQL)

### Step 1: Create docker-compose.yml

```yaml
name: featbit
services:
  ui:
    image: featbit/featbit-ui:latest
    environment:
      - API_URL=http://localhost:5000
      - DEMO_URL=https://featbit-samples.vercel.app
      - EVALUATION_URL=http://localhost:5100
      - BASE_HREF=/
    depends_on:
      - api-server
    ports:
      - "8081:80"
    networks:
      - featbit-network

  api-server:
    image: featbit/featbit-api-server:latest
    environment:
      - DbProvider=Postgres
      - MqProvider=Postgres
      - CacheProvider=None
      - Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password=please_change_me;Database=featbit
      - OLAP__ServiceHost=http://da-server
    depends_on:
      - postgresql
      - da-server
    ports:
      - "5000:5000"
    networks:
      - featbit-network

  evaluation-server:
    image: featbit/featbit-evaluation-server:latest
    environment:
      - DbProvider=Postgres
      - MqProvider=Postgres
      - CacheProvider=None
      - Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password=please_change_me;Database=featbit
    depends_on:
      - postgresql
    ports:
      - "5100:5100"
    networks:
      - featbit-network

  da-server:
    image: featbit/featbit-data-analytics-server:latest
    depends_on:
      - postgresql
    ports:
      - "8200:80"
    networks:
      - featbit-network
    environment:
      DB_PROVIDER: Postgres
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: please_change_me
      POSTGRES_HOST: postgresql
      POSTGRES_PORT: 5432
      POSTGRES_DATABASE: featbit
      CHECK_DB_LIVNESS: true

  postgresql:
    image: postgres:15.10
    container_name: postgresql
    restart: on-failure
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: please_change_me
    volumes:
      - postgres:/var/lib/postgresql/data
    networks:
      - featbit-network

volumes:
  postgres:

networks:
  featbit-network:
    name: featbit-network
    driver: bridge
```

### Step 2: Deploy

```bash
# Start all services
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs -f

# Stop services
docker compose down

# Stop and remove volumes (data will be lost)
docker compose down -v
```

### Step 3: Access FeatBit

- **URL**: http://localhost:8081
- **Default Credentials**: 
  - Email: `admin@featbit.com`
  - Password: `123456`

**Important**: Change the default password immediately after first login!

## Standard Deployment (Production-Ready)

For production use with MongoDB and Redis:

```yaml
name: featbit
services:
  ui:
    image: featbit/featbit-ui:latest
    environment:
      - API_URL=http://localhost:5000
      - EVALUATION_URL=http://localhost:5100
    ports:
      - "8081:80"

  api-server:
    image: featbit/featbit-api-server:latest
    environment:
      - DbProvider=MongoDb
      - MqProvider=Redis
      - CacheProvider=Redis
      - MongoDb__ConnectionString=mongodb://mongodb:27017
      - MongoDb__Database=featbit
      - Redis__ConnectionString=redis:6379
      - OLAP__ServiceHost=http://da-server
    depends_on:
      - mongodb
      - redis
    ports:
      - "5000:5000"

  evaluation-server:
    image: featbit/featbit-evaluation-server:latest
    environment:
      - DbProvider=MongoDb
      - MqProvider=Redis
      - CacheProvider=Redis
      - MongoDb__ConnectionString=mongodb://mongodb:27017
      - MongoDb__Database=featbit
      - Redis__ConnectionString=redis:6379
    depends_on:
      - mongodb
      - redis
    ports:
      - "5100:5100"

  da-server:
    image: featbit/featbit-data-analytics-server:latest
    environment:
      - MongoDb__ConnectionString=mongodb://mongodb:27017
      - MongoDb__Database=featbit
    depends_on:
      - mongodb
    ports:
      - "8200:80"

  mongodb:
    image: mongo:6.0
    ports:
      - "27017:27017"
    volumes:
      - mongodb:/data/db

  redis:
    image: redis:7.0-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis:/data

volumes:
  mongodb:
  redis:

networks:
  default:
    name: featbit-network
```

## Professional Deployment (High-Volume)

For enterprise scale with ClickHouse and Kafka:

**Note**: This requires significant infrastructure. See https://github.com/featbit/featbit for complete configuration.

Key additions:
- **ClickHouse**: For high-performance analytics
- **Kafka**: For event streaming and processing
- **Zookeeper**: For Kafka coordination

## Environment Variables Reference

### Common Variables (All Services)

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `DbProvider` | Database type: `Postgres` or `MongoDb` | - | Yes |
| `MqProvider` | Message queue: `Postgres`, `Redis`, or `Kafka` | - | Yes |
| `CacheProvider` | Cache provider: `None` or `Redis` | `None` | No |

### API Server

| Variable | Description |
|----------|-------------|
| `Postgres__ConnectionString` | PostgreSQL connection string |
| `MongoDb__ConnectionString` | MongoDB connection string |
| `MongoDb__Database` | MongoDB database name |
| `Redis__ConnectionString` | Redis connection string |
| `OLAP__ServiceHost` | Data analytics service URL |

### Evaluation Server

| Variable | Description |
|----------|-------------|
| `Postgres__ConnectionString` | PostgreSQL connection string |
| `MongoDb__ConnectionString` | MongoDB connection string |
| `MongoDb__Database` | MongoDB database name |
| `Redis__ConnectionString` | Redis connection string |

### UI

| Variable | Description |
|----------|-------------|
| `API_URL` | API server URL (must be accessible from browser) |
| `EVALUATION_URL` | Evaluation server URL (must be accessible from browser) |
| `DEMO_URL` | Demo application URL |

### Data Analytics Server

| Variable | Description |
|----------|-------------|
| `DB_PROVIDER` | Database type: `Postgres` or `MongoDb` |
| `POSTGRES_HOST` | PostgreSQL hostname |
| `POSTGRES_PORT` | PostgreSQL port |
| `POSTGRES_USER` | PostgreSQL username |
| `POSTGRES_PASSWORD` | PostgreSQL password |
| `POSTGRES_DATABASE` | PostgreSQL database name |
| `MongoDb__ConnectionString` | MongoDB connection string |
| `MongoDb__Database` | MongoDB database name |

## Troubleshooting

### Port Conflicts

If ports are already in use:

```yaml
ports:
  - "9081:80"  # Change 8081 to 9081
```

### Making FeatBit Publicly Accessible

Update UI environment variables:

```yaml
ui:
  environment:
    - API_URL=https://api.yourdomain.com
    - EVALUATION_URL=https://eval.yourdomain.com
```

### Database Connection Issues

1. Check connection string format
2. Verify network connectivity
3. Ensure database is ready before services start:

```yaml
depends_on:
  postgresql:
    condition: service_healthy
```

### Container Fails to Start

```bash
# Check logs
docker compose logs service-name

# Restart specific service
docker compose restart service-name

# Rebuild and restart
docker compose up -d --force-recreate service-name
```

## Terraform Deployment (AWS)

For automated AWS deployment:

```bash
git clone https://github.com/featbit/featbit-terraform-aws
cd featbit-terraform-aws
terraform init
terraform plan
terraform apply
```

See: https://github.com/featbit/featbit-terraform-aws

## Custom Infrastructure

If using existing infrastructure (managed databases, Redis, etc.):

1. **Set up databases**:
   - PostgreSQL 12+ or MongoDB 4.4+
   - Redis 6.0+ (optional, for Standard/Professional)
   - ClickHouse 21+ (optional, for Professional)
   - Kafka 2.8+ (optional, for Professional)

2. **Deploy services**: Use Docker or native binaries

3. **Configure environment variables**: Point services to your infrastructure

4. **Initialize databases**: Run seed scripts from https://github.com/featbit/featbit

## High Availability Considerations

### 1. Multiple Instances
- Run multiple instances of API Server and Evaluation Server
- Use load balancer (nginx, HAProxy, cloud LB)

### 2. Database HA
- PostgreSQL: Use replication or managed service (RDS, Azure Database)
- MongoDB: Use replica sets or MongoDB Atlas
- Redis: Use Redis Sentinel or managed service

### 3. Message Queue HA
- Kafka: Use multi-broker cluster
- Redis: Use Redis Cluster or managed service

### 4. Health Checks

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

## Backup and Recovery

### Database Backups

**PostgreSQL**:
```bash
docker exec postgresql pg_dump -U postgres featbit > backup.sql
docker exec -i postgresql psql -U postgres featbit < backup.sql
```

**MongoDB**:
```bash
docker exec mongodb mongodump --out /backup
docker exec mongodb mongorestore /backup
```

### Volume Backups

```bash
# Stop services
docker compose down

# Backup volumes
docker run --rm -v featbit_postgres:/data -v $(pwd):/backup alpine tar czf /backup/postgres-backup.tar.gz /data

# Restore volumes
docker run --rm -v featbit_postgres:/data -v $(pwd):/backup alpine tar xzf /backup/postgres-backup.tar.gz -C /
```

## Security Best Practices

1. **Change Default Passwords**: Update `POSTGRES_PASSWORD` and admin account
2. **Use Secrets Management**: Store credentials in Docker secrets or vault
3. **Network Isolation**: Use private networks, expose only necessary ports
4. **TLS/SSL**: Use reverse proxy (nginx) with SSL certificates
5. **Firewall Rules**: Restrict access to database and Redis ports
6. **Regular Updates**: Keep Docker images updated

## Performance Tuning

### PostgreSQL

```yaml
postgresql:
  command: postgres -c max_connections=200 -c shared_buffers=256MB
```

### Redis

```yaml
redis:
  command: redis-server --maxmemory 512mb --maxmemory-policy allkeys-lru
```

### Evaluation Server (Horizontal Scaling)

```yaml
evaluation-server:
  deploy:
    replicas: 3
```

## Kubernetes Deployment (Helm Charts)

### Prerequisites
- Kubernetes cluster >= 1.23
- Helm >= 3.7.0
- kubectl configured to connect to your cluster

### Quick Start with Helm

**Step 1: Add FeatBit Helm Repository**
```bash
helm repo add featbit https://featbit.github.io/featbit-charts/
helm repo update
```

**Step 2: Install FeatBit (Standard Edition with PostgreSQL)**
```bash
# Create namespace
kubectl create namespace featbit

# Install with default values (development/testing)
helm install featbit featbit/featbit --namespace featbit

# Install with custom values (production)
helm install featbit featbit/featbit \
  -f your-values.yaml \
  --namespace featbit
```

**Step 3: Verify Installation**
```bash
# Check pods status
kubectl get pods -n featbit

# Check services
kubectl get svc -n featbit
```

### Service Exposure Options

FeatBit requires exposing three services:
- **UI** (port 8081): Frontend interface
- **API** (port 5000): API server
- **Evaluation Server** (port 5100): Real-time feature flag evaluation

**Option 1: Ingress (Recommended for Production)**
```yaml
apiExternalUrl: "https://api.example.com"
evaluationServerExternalUrl: "https://eval.example.com"

ui:
  ingress:
    enabled: true
    host: app.example.com
    tls:
      enabled: true
      secretName: featbit-tls-secret

api:
  ingress:
    enabled: true
    host: api.example.com
    tls:
      enabled: true
      secretName: featbit-tls-secret

els:
  ingress:
    enabled: true
    host: eval.example.com
    tls:
      enabled: true
      secretName: featbit-tls-secret
```

**Option 2: LoadBalancer (Cloud Environments)**
```yaml
apiExternalUrl: "http://API_EXTERNAL_IP:5000"
evaluationServerExternalUrl: "http://ELS_EXTERNAL_IP:5100"

ui:
  service:
    type: LoadBalancer

api:
  service:
    type: LoadBalancer

els:
  service:
    type: LoadBalancer
```

**Option 3: NodePort (Development/Testing)**
```yaml
apiExternalUrl: "http://NODE_IP:30050"
evaluationServerExternalUrl: "http://NODE_IP:30100"

ui:
  service:
    type: NodePort
    nodePort: 30025

api:
  service:
    type: NodePort
    nodePort: 30050

els:
  service:
    type: NodePort
    nodePort: 30100
```

**Option 4: Port Forward (Local Development)**
```bash
# UI
kubectl port-forward service/featbit-ui 8081:8081 -n featbit

# API
kubectl port-forward service/featbit-api 5000:5000 -n featbit

# Evaluation Server
kubectl port-forward service/featbit-els 5100:5100 -n featbit
```

### Key Configuration

**Using External PostgreSQL**
```yaml
postgresql:
  enabled: false

externalPostgresql:
  host: your-postgres-server.com
  port: 5432
  user: postgres
  database: featbit
  existingSecret: featbit-db-secret
  existingSecretPasswordKey: password
```

**Using External Redis**
```yaml
redis:
  enabled: false

externalRedis:
  host: your-redis-server.com
  port: 6379
  existingSecret: featbit-redis-secret
  existingSecretPasswordKey: password
```

**Architecture Tier Selection**
```yaml
architecture:
  tier: "standard"  # Options: "standalone", "standard", "professional"
  database: "postgres"  # Options: "postgres", "mongodb"
```

**Auto-Scaling**
```yaml
api:
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 10
    targetCPUUtilizationPercentage: 80

els:
  autoscaling:
    enabled: true
    minReplicas: 3
    maxReplicas: 20
    targetCPUUtilizationPercentage: 75
```

### Production Deployment Example (Azure AKS)

The Helm chart repository includes a comprehensive AKS deployment guide with:
- NGINX Ingress Controller with TLS
- cert-manager for automatic Let's Encrypt certificates
- Azure Key Vault integration for secrets
- Multi-region setup with Azure Traffic Manager

**Example values for AKS:**
```yaml
apiExternalUrl: "https://api.featbit.example.com"
evaluationServerExternalUrl: "https://eval.featbit.example.com"

# Use external Azure Database for PostgreSQL
postgresql:
  enabled: false
externalPostgresql:
  host: featbit-pg.postgres.database.azure.com
  port: 5432
  user: featbitadmin
  database: featbit
  existingSecret: azure-kv-postgres-secret

# Use Azure Cache for Redis
redis:
  enabled: false
externalRedis:
  host: featbit-redis.redis.cache.windows.net
  port: 6380
  tls: true
  existingSecret: azure-kv-redis-secret

# Ingress with TLS
global:
  ingressClassName: nginx

ui:
  ingress:
    enabled: true
    host: app.featbit.example.com
    tls:
      enabled: true
      secretName: featbit-tls-cert
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-prod"

api:
  ingress:
    enabled: true
    host: api.featbit.example.com
    tls:
      enabled: true
      secretName: featbit-tls-cert
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-prod"

els:
  ingress:
    enabled: true
    host: eval.featbit.example.com
    tls:
      enabled: true
      secretName: featbit-tls-cert
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-prod"
```

### Upgrade and Migration

**Upgrading to Latest Version**
```bash
# Update Helm repo
helm repo update featbit

# Check available versions
helm search repo featbit/featbit --versions

# Upgrade (always backup database first!)
helm upgrade featbit featbit/featbit \
  --version <target-version> \
  -f your-values.yaml \
  --namespace featbit
```

**Important Notes:**
- Always backup your database before upgrading
- Review migration scripts in the chart's `migration/` folder
- For external databases, manually apply migration scripts
- Test upgrades in non-production environment first

### Helm Chart Resources

- **Chart Repository**: https://github.com/featbit/featbit-charts
- **Chart Documentation**: https://github.com/featbit/featbit-charts/blob/main/README.md
- **Examples**: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples
- **AKS Deployment Guide**: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples/aks
- **Current Version**: 0.9.0 (App: FeatBit 5.2.0)

For detailed Helm configuration options and production deployment patterns, refer to the official chart repository.

## Monitoring

Add Prometheus and Grafana for observability:

```yaml
prometheus:
  image: prom/prometheus:latest
  volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml
  ports:
    - "9090:9090"

grafana:
  image: grafana/grafana:latest
  ports:
    - "3000:3000"
```

## Official Resources

- **Main Repository**: https://github.com/featbit/featbit
- **Helm Charts**: https://github.com/featbit/featbit-charts
- **Terraform AWS**: https://github.com/featbit/featbit-terraform-aws
- **Documentation**: https://docs.featbit.co/installation
- **Docker Hub**: https://hub.docker.com/u/featbit
