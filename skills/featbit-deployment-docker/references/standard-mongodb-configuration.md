---
name: standard-mongodb-configuration
description: Complete docker-compose configuration for FeatBit Standard deployment with MongoDB + Redis, including initialization scripts and health checks.
---

# Standard Deployment - MongoDB + Redis

## Table of Contents

1. [Overview](#overview)
2. [Complete Configuration](#complete-configuration)
3. [MongoDB Initialization](#mongodb-initialization)
4. [Deployment Steps](#deployment-steps)
5. [Resource Requirements](#resource-requirements)
6. [Reference](#reference)

## Overview

Standard deployment with MongoDB + Redis provides:
- **Better performance**: Redis caching reduces database load
- **Improved reliability**: Dedicated message queue via Redis
- **Production-ready**: Suitable for medium to large teams
- **Scalability**: Can handle high-volume traffic
- **Document-oriented**: Better for flexible schema workloads

## Complete Configuration

**Source file**: `docker-compose-mongodb.yml` in the FeatBit repository.

```yaml
name: featbit
services:
  ui:
    image: featbit/featbit-ui:latest
    environment:
      - API_URL=http://localhost:5000
      - EVALUATION_URL=http://localhost:5100
      - DEMO_URL=https://featbit-samples.vercel.app
    ports:
      - "8081:80"
    networks:
      - featbit-network

  api-server:
    image: featbit/featbit-api-server:latest
    environment:
      - DbProvider=MongoDb
      - MqProvider=Redis
      - CacheProvider=Redis
      - MongoDb__ConnectionString=mongodb://admin:please_change_me@mongodb:27017
      - MongoDb__Database=featbit
      - Redis__ConnectionString=redis:6379
      - OLAP__ServiceHost=http://da-server
    depends_on:
      - mongodb
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
      - DbProvider=MongoDb
      - MqProvider=Redis
      - CacheProvider=Redis
      - MongoDb__ConnectionString=mongodb://admin:please_change_me@mongodb:27017
      - MongoDb__Database=featbit
      - Redis__ConnectionString=redis:6379
    depends_on:
      - mongodb
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
    environment:
      - DB_PROVIDER=MongoDb
      - MongoDb__ConnectionString=mongodb://admin:please_change_me@mongodb:27017
      - MongoDb__Database=featbit
    depends_on:
      - mongodb
    ports:
      - "8200:80"
    networks:
      - featbit-network

  mongodb:
    image: mongo:6.0
    container_name: mongodb
    restart: on-failure
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: please_change_me
      MONGO_INITDB_DATABASE: featbit
    volumes:
      - mongodb:/data/db
      - ./infra/mongodb/docker-entrypoint-initdb.d/:/docker-entrypoint-initdb.d/
    networks:
      - featbit-network
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
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
  mongodb:
  redis:

networks:
  featbit-network:
    name: featbit-network
    driver: bridge
```

## MongoDB Initialization

**Automatic Initialization**: MongoDB initialization scripts are automatically executed from the cloned repository.

The volume mapping in the configuration:
```yaml
volumes:
  - ./infra/mongodb/docker-entrypoint-initdb.d/:/docker-entrypoint-initdb.d/
```

This automatically mounts all initialization scripts (*.js files) from the repository to the MongoDB container. All scripts are executed in alphabetical order (v0.0.0.js, v1.0.0.js, etc.) on first container startup.

**Scripts Location**: https://github.com/featbit/featbit/tree/main/infra/mongodb/docker-entrypoint-initdb.d

**Important**: The scripts will only run on the **first startup** when the database is empty. If you need to re-initialize, remove the MongoDB volume:
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
docker compose -f docker-compose-mongodb.yml up -d
```

### Customize Configuration

```bash
# Create .env file for secrets
cat > .env <<EOF
MONGO_PASSWORD=your_strong_mongo_password
REDIS_PASSWORD=your_redis_password_if_needed
EOF

# Update docker-compose.yml to use environment variables
# Replace hardcoded passwords with ${MONGO_PASSWORD}
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
- **MongoDB Configuration**: https://github.com/featbit/featbit/blob/main/docker-compose-mongodb.yml
- **MongoDB Init Script**: https://github.com/featbit/featbit/blob/main/infra/mongodb/docker-entrypoint-initdb.d/v0.0.0.js
- **Deployment Options Comparison**: https://docs.featbit.co/installation/deployment-options
