# Standard Deployment - Complete Configuration

Complete configurations for FeatBit Standard deployment with PostgreSQL or MongoDB + Redis.

## Architecture Overview

Standard deployment adds Redis for caching and message queuing, providing:
- **Better performance**: Redis caching reduces database load
- **Improved reliability**: Dedicated message queue
- **Production-ready**: Suitable for medium to large teams
- **Scalability**: Can handle high-volume traffic

## Option A: Standard with PostgreSQL + Redis

### Complete docker-compose.yml

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

## Option B: Standard with MongoDB + Redis

### Complete docker-compose.yml

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
    volumes:
      - mongodb:/data/db
      - ./infra/mongodb/docker-entrypoint-initdb.d:/docker-entrypoint-initdb.d:ro
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

### MongoDB Initialization

**Important**: MongoDB requires initialization script for default data.

```bash
# Create directory structure
mkdir -p infra/mongodb/docker-entrypoint-initdb.d

# Download initialization script
curl -o infra/mongodb/docker-entrypoint-initdb.d/v0.0.0.js \
  https://raw.githubusercontent.com/featbit/featbit/main/infra/mongodb/docker-entrypoint-initdb.d/v0.0.0.js
```

**Script Reference**: https://github.com/featbit/featbit/blob/main/infra/mongodb/docker-entrypoint-initdb.d/v0.0.0.js

## Deployment Steps

### Download Official Configuration

```bash
# For MongoDB + Redis configuration
curl -O https://raw.githubusercontent.com/featbit/featbit/main/docker-compose-mongodb.yml
mv docker-compose-mongodb.yml docker-compose.yml

# For PostgreSQL + Redis
# Modify the Standalone configuration to use Redis for MqProvider and CacheProvider
```

### Customize Configuration

```bash
# Create .env file for secrets
cat > .env <<EOF
DB_PASSWORD=your_strong_password_here
MONGO_PASSWORD=your_strong_mongo_password
REDIS_PASSWORD=your_redis_password_if_needed
EOF

# Update docker-compose.yml to use environment variables
# Replace hardcoded passwords with ${DB_PASSWORD}
```

### Deploy

```bash
docker compose up -d
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
    # Same as api-server
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

## When to Choose Standard

✅ **Use Standard When**:
- Moving to production
- Team size: 10-50 users
- Expected traffic: moderate to high
- Need better performance than Standalone
- Require production-level reliability
- Need caching for faster responses

⚠️ **Consider Professional When**:
- Team size: >50 users
- Very high traffic (millions of requests/day)
- Need advanced analytics with ClickHouse
- Require event streaming capabilities
- Need massive horizontal scalability

## Reference

- **Official Documentation**: https://docs.featbit.co/installation/docker-compose
- **MongoDB Configuration**: https://github.com/featbit/featbit/blob/main/docker-compose-mongodb.yml
- **MongoDB Init Script**: https://github.com/featbit/featbit/blob/main/infra/mongodb/docker-entrypoint-initdb.d/v0.0.0.js
- **Deployment Options Comparison**: https://docs.featbit.co/installation/deployment-options
