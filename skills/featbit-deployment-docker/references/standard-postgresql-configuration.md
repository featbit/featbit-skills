---
name: standard-postgresql-configuration
description: Complete docker-compose configuration for FeatBit Standard deployment with PostgreSQL + Redis, including initialization scripts and health checks.
---

# Standard Deployment - PostgreSQL + Redis

## Table of Contents

1. [Overview](#overview)
2. [Complete Configuration](#complete-configuration)
3. [PostgreSQL Initialization](#postgresql-initialization)
4. [Deployment Steps](#deployment-steps)
5. [Using Managed Services](#using-managed-services)
6. [Resource Requirements](#resource-requirements)
7. [Reference](#reference)

## Overview

Standard deployment with PostgreSQL + Redis provides:
- **Better performance**: Redis caching reduces database load
- **Improved reliability**: Dedicated message queue via Redis
- **Production-ready**: Suitable for medium to large teams
- **Scalability**: Can handle high-volume traffic
- **Familiarity**: PostgreSQL is widely known and well-supported

## Complete Configuration

**Source file**: `docker-compose-standard.yml` in the FeatBit repository.

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
      - MqProvider=Redis
      - CacheProvider=Redis
      - Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password=please_change_me;Database=featbit
      - Redis__ConnectionString=redis:6379
      - OLAP__ServiceHost=http://da-server
    depends_on:
      - postgresql
      - redis
      - da-server
    ports:
      - "5000:5000"
    networks:
      - featbit-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health/liveness"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  evaluation-server:
    image: featbit/featbit-evaluation-server:latest
    environment:
      - DbProvider=Postgres
      - MqProvider=Redis
      - CacheProvider=Redis
      - Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password=please_change_me;Database=featbit
      - Redis__ConnectionString=redis:6379
    depends_on:
      - postgresql
      - redis
    ports:
      - "5100:5100"
    networks:
      - featbit-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5100/health/liveness"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

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
      - ./infra/postgresql/docker-entrypoint-initdb.d/:/docker-entrypoint-initdb.d/
    networks:
      - featbit-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7.0-alpine
    container_name: redis
    restart: on-failure
    ports:
      - "6379:6379"
    volumes:
      - redis:/data
    networks:
      - featbit-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    command: >
      redis-server
      --maxmemory 512mb
      --maxmemory-policy allkeys-lru
      --save 60 1000
      --appendonly yes

volumes:
  postgres:
  redis:

networks:
  featbit-network:
    name: featbit-network
    driver: bridge
```

## PostgreSQL Initialization

**Automatic Initialization**: PostgreSQL initialization scripts are automatically executed from the cloned repository.

The volume mapping in the configuration:
```yaml
volumes:
  - ./infra/postgresql/docker-entrypoint-initdb.d/:/docker-entrypoint-initdb.d/
```

This automatically mounts all initialization scripts from the repository to the PostgreSQL container. Scripts are executed in alphabetical order (v0.0.0.sql, v1.0.0.sql, etc.) on first container startup.

**Scripts Location**: https://github.com/featbit/featbit/tree/main/infra/postgresql/docker-entrypoint-initdb.d

**Important**: The scripts will only run on the **first startup** when the database is empty. If you need to re-initialize, remove the PostgreSQL volume:
```bash
docker compose down -v  # This removes all volumes
# or
docker volume rm <volume_name>  # Remove specific volume
```

## Deployment Steps

### Prerequisites

1. Clone the FeatBit repository (contains all configuration files and initialization scripts):
```bash
git clone https://github.com/featbit/featbit.git
cd featbit
```

2. Docker 20.10+ and Docker Compose 2.0+ installed
3. 4GB RAM minimum (8GB recommended)

### Start Services

```bash
docker compose -f docker-compose-standard.yml up -d
```

### Customize Configuration

```bash
# Create .env file for secrets
cat > .env <<EOF
DB_PASSWORD=your_strong_password_here
REDIS_PASSWORD=your_redis_password_if_needed
EOF

# Update docker-compose.yml to use environment variables
# Replace hardcoded passwords with ${DB_PASSWORD}
```

## Using Managed Services

### AWS Example

```yaml
api-server:
  environment:
    # RDS PostgreSQL
    - Postgres__ConnectionString=Host=featbit.xxxxx.us-east-1.rds.amazonaws.com;Port=5432;Username=admin;Password=${DB_PASSWORD};Database=featbit;SSL Mode=Require
    # ElastiCache Redis
    - Redis__ConnectionString=featbit-redis.xxxxx.cache.amazonaws.com:6379

evaluation-server:
  environment:
    - Postgres__ConnectionString=Host=featbit.xxxxx.us-east-1.rds.amazonaws.com;Port=5432;Username=admin;Password=${DB_PASSWORD};Database=featbit;SSL Mode=Require
    - Redis__ConnectionString=featbit-redis.xxxxx.cache.amazonaws.com:6379

# Remove postgresql and redis services if using managed
```

### Azure Example

```yaml
api-server:
  environment:
    # Azure Database for PostgreSQL
    - Postgres__ConnectionString=Host=featbit.postgres.database.azure.com;Port=5432;Username=admin@featbit;Password=${DB_PASSWORD};Database=featbit;SSL Mode=Require
    # Azure Cache for Redis
    - Redis__ConnectionString=featbit.redis.cache.windows.net:6380,password=${REDIS_PASSWORD},ssl=True
```

## Resource Requirements

**Minimum**:
- RAM: 4GB
- CPU: 2 cores
- Disk: 20GB

**Recommended for Production**:
- RAM: 8GB
- CPU: 4 cores
- Disk: 50GB+
- SSD storage

## Reference

- **Official Documentation**: https://docs.featbit.co/installation/docker-compose
- **Deployment Options Comparison**: https://docs.featbit.co/installation/deployment-options
- **PostgreSQL Init Scripts**: https://github.com/featbit/featbit/tree/main/infra/postgresql/docker-entrypoint-initdb.d
