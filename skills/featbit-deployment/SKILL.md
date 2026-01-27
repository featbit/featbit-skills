---
name: featbit-deployment
description: Expert guidance for deploying FeatBit using Docker Compose, including standalone, standard, and professional deployment options. Use when users ask about deploying, installing, or running FeatBit infrastructure.
appliesTo:
  - "**/docker-compose.yml"
  - "**/docker-compose.yaml"
  - "**/*.tf"
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
- **Terraform AWS**: https://github.com/featbit/featbit-terraform-aws
- **Documentation**: https://docs.featbit.co/installation
- **Docker Hub**: https://hub.docker.com/u/featbit
