---
name: featbit-deployment-docker
description: Expert guidance for deploying FeatBit using Docker Compose. Covers standalone, standard, and professional deployment tiers with Docker. Use when users work with docker-compose files or ask about Docker-based FeatBit deployments.
appliesTo:
  - "**/docker-compose.yml"
  - "**/docker-compose.yaml"
  - "**/Dockerfile"
---

# FeatBit Docker Compose Deployment Expert

You are an expert on deploying FeatBit using Docker Compose, with deep knowledge of container orchestration, environment configuration, and production-ready Docker deployments.

## When to Use This Skill

Activate when users:
- Work with `docker-compose.yml` files
- Ask about Docker-based FeatBit deployment
- Need help with container configuration
- Want to deploy FeatBit on a single server
- Need guidance on Docker networking and volumes
- Ask about local development setup

## Deployment Tiers Overview

FeatBit offers three Docker Compose deployment tiers:

### 1. Standalone (Minimal)
- **Database**: PostgreSQL only
- **Cache**: None
- **Message Queue**: PostgreSQL
- **Analytics**: PostgreSQL
- **Best For**: Development, testing, small teams
- **Pros**: Minimal infrastructure, easy setup, quick start
- **Cons**: Limited scalability, single point of failure

### 2. Standard (Balanced)
- **Database**: PostgreSQL or MongoDB
- **Cache**: Redis
- **Message Queue**: Redis
- **Analytics**: MongoDB
- **Best For**: Production use, medium teams (up to 50 users)
- **Pros**: Good scalability, reliable performance, battle-tested
- **Cons**: More infrastructure to manage

### 3. Professional (High-Volume)
- **Database**: PostgreSQL + MongoDB + ClickHouse
- **Cache**: Redis
- **Message Queue**: Kafka
- **Analytics**: ClickHouse
- **Best For**: Large enterprises, high traffic (1000+ users)
- **Pros**: Massive scalability, advanced analytics, event streaming
- **Cons**: Most complex setup, significant resource requirements

## Quick Start: Standalone Deployment

### Prerequisites
- Docker 20.10+ and Docker Compose 2.0+
- 2GB RAM minimum (4GB recommended)
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

# View specific service logs
docker compose logs -f api-server

# Stop services
docker compose down

# Stop and remove volumes (data will be lost!)
docker compose down -v
```

### Step 3: Access FeatBit

- **URL**: http://localhost:8081
- **Default Credentials**: 
  - Email: `test@featbit.com`
  - Password: `123456`

**⚠️ Important**: Change the default password immediately after first login!

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
      - MongoDb__ConnectionString=mongodb://mongodb:27017
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
    networks:
      - featbit-network

  da-server:
    image: featbit/featbit-data-analytics-server:latest
    environment:
      - DB_PROVIDER=MongoDb
      - MongoDb__ConnectionString=mongodb://mongodb:27017
      - MongoDb__Database=featbit
    depends_on:
      - mongodb
    ports:
      - "8200:80"
    networks:
      - featbit-network

  mongodb:
    image: mongo:6.0
    ports:
      - "27017:27017"
    volumes:
      - mongodb:/data/db
    networks:
      - featbit-network

  redis:
    image: redis:7.0-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis:/data
    networks:
      - featbit-network

volumes:
  mongodb:
  redis:

networks:
  featbit-network:
    name: featbit-network
    driver: bridge
```

## Professional Deployment (High-Volume)

For enterprise scale with ClickHouse and Kafka:

**Note**: This requires significant infrastructure resources (8GB+ RAM, 4+ CPU cores).

Complete configuration: https://github.com/featbit/featbit

Key additions:
- **ClickHouse**: High-performance columnar analytics database
- **Kafka + Zookeeper**: Event streaming and message processing
- **ClickHouse Analytics**: Advanced data warehouse capabilities

## Environment Variables Reference

### Common Variables (All Services)

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `DbProvider` | Database type: `Postgres` or `MongoDb` | - | Yes |
| `MqProvider` | Message queue: `Postgres`, `Redis`, or `Kafka` | - | Yes |
| `CacheProvider` | Cache provider: `None` or `Redis` | `None` | No |

### API Server

| Variable | Description | Example |
|----------|-------------|---------|
| `Postgres__ConnectionString` | PostgreSQL connection string | `Host=postgresql;Port=5432;Username=postgres;Password=***;Database=featbit` |
| `MongoDb__ConnectionString` | MongoDB connection string | `mongodb://mongodb:27017` |
| `MongoDb__Database` | MongoDB database name | `featbit` |
| `Redis__ConnectionString` | Redis connection string | `redis:6379` |
| `OLAP__ServiceHost` | Data analytics service URL | `http://da-server` |

### Evaluation Server

| Variable | Description | Example |
|----------|-------------|---------|
| `Postgres__ConnectionString` | PostgreSQL connection string | Same as API Server |
| `MongoDb__ConnectionString` | MongoDB connection string | Same as API Server |
| `MongoDb__Database` | MongoDB database name | `featbit` |
| `Redis__ConnectionString` | Redis connection string | `redis:6379` |

### UI

| Variable | Description | Example |
|----------|-------------|---------|
| `API_URL` | API server URL (accessible from browser) | `http://localhost:5000` or `https://api.yourdomain.com` |
| `EVALUATION_URL` | Evaluation server URL (accessible from browser) | `http://localhost:5100` or `https://eval.yourdomain.com` |
| `DEMO_URL` | Demo application URL | `https://featbit-samples.vercel.app` |
| `BASE_HREF` | Base URL path | `/` (or `/featbit/` for sub-path) |

### Data Analytics Server

| Variable | Description | Example |
|----------|-------------|---------|
| `DB_PROVIDER` | Database type: `Postgres` or `MongoDb` | `Postgres` |
| `POSTGRES_HOST` | PostgreSQL hostname | `postgresql` |
| `POSTGRES_PORT` | PostgreSQL port | `5432` |
| `POSTGRES_USER` | PostgreSQL username | `postgres` |
| `POSTGRES_PASSWORD` | PostgreSQL password | `your_password` |
| `POSTGRES_DATABASE` | PostgreSQL database name | `featbit` |
| `MongoDb__ConnectionString` | MongoDB connection string | `mongodb://mongodb:27017` |
| `MongoDb__Database` | MongoDB database name | `featbit` |

## Production Configurations

### Using Managed Databases

Replace embedded databases with managed services:

```yaml
api-server:
  environment:
    # Use AWS RDS PostgreSQL
    - Postgres__ConnectionString=Host=featbit.xxxxx.us-east-1.rds.amazonaws.com;Port=5432;Username=admin;Password=${DB_PASSWORD};Database=featbit;SSL Mode=Require
    
    # Use Azure Cache for Redis
    - Redis__ConnectionString=featbit.redis.cache.windows.net:6380,password=${REDIS_PASSWORD},ssl=True
```

### Reverse Proxy with nginx

```yaml
nginx:
  image: nginx:alpine
  ports:
    - "80:80"
    - "443:443"
  volumes:
    - ./nginx.conf:/etc/nginx/nginx.conf:ro
    - ./ssl:/etc/nginx/ssl:ro
  depends_on:
    - ui
    - api-server
    - evaluation-server
  networks:
    - featbit-network
```

Example `nginx.conf`:

```nginx
server {
    listen 443 ssl;
    server_name app.yourdomain.com;
    
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    
    location / {
        proxy_pass http://ui:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

server {
    listen 443 ssl;
    server_name api.yourdomain.com;
    
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    
    location / {
        proxy_pass http://api-server:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

server {
    listen 443 ssl;
    server_name eval.yourdomain.com;
    
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    
    location / {
        proxy_pass http://evaluation-server:5100;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        
        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Health Checks

```yaml
api-server:
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:5000/health/liveness"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 40s

evaluation-server:
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:5100/health/liveness"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 40s
```

### Resource Limits

```yaml
api-server:
  deploy:
    resources:
      limits:
        cpus: '2'
        memory: 2G
      reservations:
        cpus: '1'
        memory: 512M
```

## Troubleshooting

### Port Conflicts

If ports are already in use, change the host port:

```yaml
ui:
  ports:
    - "9081:80"  # Changed from 8081 to 9081
```

### Making FeatBit Publicly Accessible

Update UI environment variables to use public URLs:

```yaml
ui:
  environment:
    - API_URL=https://api.yourdomain.com
    - EVALUATION_URL=https://eval.yourdomain.com
```

**Important**: Ensure API and Evaluation Server URLs are accessible from users' browsers, not just from the Docker network.

### Database Connection Issues

1. **Check connection string format** - Ensure no typos
2. **Verify network connectivity** - All services must be on same network
3. **Ensure database is ready** - Use `depends_on` with health checks:

```yaml
api-server:
  depends_on:
    postgresql:
      condition: service_healthy

postgresql:
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U postgres"]
    interval: 5s
    timeout: 5s
    retries: 5
```

### Container Fails to Start

```bash
# Check logs
docker compose logs api-server

# Check specific service status
docker compose ps api-server

# Restart specific service
docker compose restart api-server

# Rebuild and restart
docker compose up -d --force-recreate api-server

# Remove everything and start fresh
docker compose down -v
docker compose up -d
```

### Disk Space Issues

```bash
# Check Docker disk usage
docker system df

# Clean up unused images and volumes
docker system prune -a --volumes
```

## Backup and Recovery

### Database Backups

**PostgreSQL**:
```bash
# Backup
docker exec postgresql pg_dump -U postgres featbit > backup_$(date +%Y%m%d).sql

# Restore
docker exec -i postgresql psql -U postgres featbit < backup_20260128.sql
```

**MongoDB**:
```bash
# Backup
docker exec mongodb mongodump --out /backup --db featbit
docker cp mongodb:/backup ./mongodb-backup

# Restore
docker cp ./mongodb-backup mongodb:/backup
docker exec mongodb mongorestore /backup/featbit --db featbit
```

### Volume Backups

```bash
# Stop services first
docker compose down

# Backup PostgreSQL volume
docker run --rm \
  -v featbit_postgres:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres-$(date +%Y%m%d).tar.gz /data

# Restore PostgreSQL volume
docker run --rm \
  -v featbit_postgres:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/postgres-20260128.tar.gz -C /
```

## Security Best Practices

### 1. Change Default Credentials

```yaml
postgresql:
  environment:
    POSTGRES_PASSWORD: ${DB_PASSWORD}  # Use secrets

# Create .env file
echo "DB_PASSWORD=strong_random_password" > .env
```

### 2. Use Docker Secrets (Swarm Mode)

```yaml
services:
  api-server:
    secrets:
      - db_password
    environment:
      - Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password_File=/run/secrets/db_password;Database=featbit

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### 3. Network Isolation

```yaml
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access

services:
  ui:
    networks:
      - frontend
  
  api-server:
    networks:
      - frontend
      - backend
  
  postgresql:
    networks:
      - backend  # Only accessible from backend network
```

### 4. TLS/SSL Certificates

Use Let's Encrypt with nginx or Traefik reverse proxy for automatic SSL certificates.

### 5. Firewall Rules

```bash
# Allow only necessary ports
ufw allow 80/tcp
ufw allow 443/tcp
ufw deny 5432/tcp  # Block direct PostgreSQL access
ufw deny 27017/tcp # Block direct MongoDB access
```

## Performance Tuning

### PostgreSQL Optimization

```yaml
postgresql:
  command: >
    postgres
    -c max_connections=200
    -c shared_buffers=256MB
    -c effective_cache_size=1GB
    -c maintenance_work_mem=64MB
    -c checkpoint_completion_target=0.9
    -c wal_buffers=16MB
    -c default_statistics_target=100
    -c random_page_cost=1.1
    -c effective_io_concurrency=200
```

### Redis Optimization

```yaml
redis:
  command: >
    redis-server
    --maxmemory 512mb
    --maxmemory-policy allkeys-lru
    --save 60 1000
    --appendonly yes
```

### MongoDB Optimization

```yaml
mongodb:
  command: >
    mongod
    --wiredTigerCacheSizeGB 1
    --maxConns 200
```

### Horizontal Scaling (Docker Swarm)

```yaml
evaluation-server:
  deploy:
    replicas: 3
    update_config:
      parallelism: 1
      delay: 10s
    restart_policy:
      condition: on-failure
```

## Monitoring

### Add Prometheus and Grafana

```yaml
prometheus:
  image: prom/prometheus:latest
  volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
  ports:
    - "9090:9090"
  networks:
    - featbit-network

grafana:
  image: grafana/grafana:latest
  environment:
    - GF_SECURITY_ADMIN_PASSWORD=admin
  ports:
    - "3000:3000"
  networks:
    - featbit-network
```

### Container Metrics with cAdvisor

```yaml
cadvisor:
  image: gcr.io/cadvisor/cadvisor:latest
  ports:
    - "8080:8080"
  volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:ro
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
  networks:
    - featbit-network
```

## Upgrade Strategy

### Rolling Updates

```bash
# Pull latest images
docker compose pull

# Recreate services with new images
docker compose up -d

# Check logs
docker compose logs -f
```

### Blue-Green Deployment

Use different compose files for zero-downtime updates:

```bash
# Start new version
docker compose -f docker-compose-v2.yml up -d

# Switch traffic (update nginx config)
# Then remove old version
docker compose -f docker-compose-v1.yml down
```

## Official Resources

### Documentation & Guides
- **Official Installation Guide**: https://docs.featbit.co/installation/docker-compose
- **Main Documentation**: https://docs.featbit.co/installation

### Source Code & Configuration Files
- **Main Repository**: https://github.com/featbit/featbit
- **Standalone docker-compose.yml**: https://github.com/featbit/featbit/blob/main/docker-compose.yml
- **Standard docker-compose.yml** (MongoDB + Redis): https://github.com/featbit/featbit/blob/main/docker-compose-mongodb.yml
- **Professional docker-compose.yml** (ClickHouse + Kafka): https://github.com/featbit/featbit/blob/main/docker-compose-pro.yml
- **MongoDB Init Script** (default credentials): https://github.com/featbit/featbit/blob/main/infra/mongodb/docker-entrypoint-initdb.d/v0.0.0.js
- **PostgreSQL Init Scripts**: https://github.com/featbit/featbit/tree/main/infra/postgresql/docker-entrypoint-initdb.d

### Docker Images
- **Docker Hub Organization**: https://hub.docker.com/u/featbit
- **UI Image**: https://hub.docker.com/r/featbit/featbit-ui
- **API Server Image**: https://hub.docker.com/r/featbit/featbit-api-server
- **Evaluation Server Image**: https://hub.docker.com/r/featbit/featbit-evaluation-server
- **Data Analytics Server Image**: https://hub.docker.com/r/featbit/featbit-data-analytics-server

### Infrastructure as Code
- **Terraform (AWS)**: https://github.com/featbit/featbit-terraform-aws

### Additional Resources
- **Sample Applications**: https://github.com/featbit/featbit-samples
- **Development Compose Files**: https://github.com/featbit/featbit/tree/main/docker/composes
