# Professional Deployment - Complete Configuration

Complete guide for FeatBit Professional deployment with ClickHouse and Kafka for enterprise-scale deployments.

## Architecture Overview

Professional deployment provides:
- **Kafka**: High-throughput message queue for event streaming
- **ClickHouse**: High-performance columnar database for analytics
- **Massive scalability**: Handle millions of events per day
- **Advanced analytics**: Real-time data warehousing
- **Production-grade**: Enterprise reliability and performance

⚠️ **Complexity Warning**: Professional tier requires significant DevOps expertise and infrastructure resources.

## System Requirements

**Minimum**:
- RAM: 8GB
- CPU: 4 cores
- Disk: 50GB SSD

**Recommended**:
- RAM: 16GB+
- CPU: 8+ cores
- Disk: 100GB+ SSD
- Network: High bandwidth

## Complete docker-compose.yml

```yaml
name: featbit-pro
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
      - DbProvider=Postgres  # or MongoDb
      - MqProvider=Kafka
      - CacheProvider=Redis
      - Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password=please_change_me;Database=featbit
      - Kafka__ConnectionString=kafka:9092
      - Redis__ConnectionString=redis:6379
      - OLAP__ServiceHost=http://da-server
    depends_on:
      - postgresql
      - kafka
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
      start_period: 60s
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G

  evaluation-server:
    image: featbit/featbit-evaluation-server:latest
    environment:
      - DbProvider=Postgres  # or MongoDb
      - MqProvider=Kafka
      - CacheProvider=Redis
      - Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password=please_change_me;Database=featbit
      - Kafka__ConnectionString=kafka:9092
      - Redis__ConnectionString=redis:6379
    depends_on:
      - postgresql
      - kafka
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
      start_period: 60s
    deploy:
      replicas: 2  # Scale horizontally
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G

  da-server:
    image: featbit/featbit-data-analytics-server:latest
    environment:
      - DB_PROVIDER=ClickHouse
      - CLICKHOUSE_HOST=clickhouse-server
      - CLICKHOUSE_PORT=8123
      - CLICKHOUSE_USER=default
      - CLICKHOUSE_PASSWORD=please_change_me
      - CLICKHOUSE_DATABASE=featbit
    depends_on:
      - clickhouse-server
    ports:
      - "8200:80"
    networks:
      - featbit-network
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G

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
    command: >
      postgres
      -c max_connections=200
      -c shared_buffers=512MB
      -c effective_cache_size=2GB
      -c maintenance_work_mem=128MB
      -c checkpoint_completion_target=0.9
      -c wal_buffers=16MB
      -c default_statistics_target=100
      -c random_page_cost=1.1
      -c effective_io_concurrency=200

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
      --maxmemory 1gb
      --maxmemory-policy allkeys-lru
      --save 60 1000
      --appendonly yes
      --appendfsync everysec

  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    container_name: zookeeper
    restart: on-failure
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    volumes:
      - zookeeper:/var/lib/zookeeper/data
    networks:
      - featbit-network
    healthcheck:
      test: ["CMD", "zkServer.sh", "status"]
      interval: 10s
      timeout: 5s
      retries: 5

  kafka:
    image: confluentinc/cp-kafka:latest
    container_name: kafka
    restart: on-failure
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "9093:9093"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: true
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: 1073741824
    volumes:
      - kafka:/var/lib/kafka/data
    networks:
      - featbit-network
    healthcheck:
      test: ["CMD", "kafka-broker-api-versions", "--bootstrap-server", "localhost:9092"]
      interval: 10s
      timeout: 10s
      retries: 5

  clickhouse-server:
    image: clickhouse/clickhouse-server:latest
    container_name: clickhouse-server
    restart: on-failure
    ports:
      - "8123:8123"
      - "9000:9000"
    environment:
      CLICKHOUSE_DB: featbit
      CLICKHOUSE_USER: default
      CLICKHOUSE_PASSWORD: please_change_me
    volumes:
      - clickhouse:/var/lib/clickhouse
    networks:
      - featbit-network
    healthcheck:
      test: ["CMD", "clickhouse-client", "--query", "SELECT 1"]
      interval: 10s
      timeout: 5s
      retries: 5
    ulimits:
      nofile:
        soft: 262144
        hard: 262144

volumes:
  postgres:
  redis:
  kafka:
  zookeeper:
  clickhouse:

networks:
  featbit-network:
    name: featbit-network
    driver: bridge
```

## Deployment Steps

### 1. Download Official Configuration

```bash
curl -O https://raw.githubusercontent.com/featbit/featbit/main/docker-compose-pro.yml
mv docker-compose-pro.yml docker-compose.yml
```

**Source**: https://github.com/featbit/featbit/blob/main/docker-compose-pro.yml

### 2. Prepare Environment

```bash
# Create .env file
cat > .env <<EOF
DB_PASSWORD=your_strong_postgres_password
CLICKHOUSE_PASSWORD=your_strong_clickhouse_password
REDIS_PASSWORD=your_redis_password_if_needed
KAFKA_PASSWORD=your_kafka_password_if_needed
EOF
```

### 3. Initialize ClickHouse

ClickHouse may require initialization scripts for schema setup:

```bash
# Create ClickHouse init directory
mkdir -p infra/clickhouse/docker-entrypoint-initdb.d

# Check official repository for init scripts
# https://github.com/featbit/featbit/tree/main/infra/clickhouse
```

### 4. Deploy

```bash
# Start infrastructure services first
docker compose up -d zookeeper kafka clickhouse-server postgresql redis

# Wait for infrastructure to be ready (30-60 seconds)
docker compose ps

# Start application services
docker compose up -d ui api-server evaluation-server da-server

# Monitor logs
docker compose logs -f
```

## Horizontal Scaling

### Scale Evaluation Server

```bash
# Scale evaluation-server to 3 replicas
docker compose up -d --scale evaluation-server=3
```

For proper load balancing, add nginx or other load balancer:

```yaml
nginx:
  image: nginx:alpine
  ports:
    - "5100:80"
  volumes:
    - ./nginx-evaluation.conf:/etc/nginx/nginx.conf:ro
  depends_on:
    - evaluation-server
  networks:
    - featbit-network
```

**nginx-evaluation.conf**:
```nginx
upstream evaluation_servers {
    server evaluation-server:5100;
    # Add more servers if scaled
}

server {
    listen 80;
    location / {
        proxy_pass http://evaluation_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## Using Managed Services

For production, strongly recommend using managed services:

### AWS Example

```yaml
api-server:
  environment:
    - Postgres__ConnectionString=Host=featbit.xxxxx.rds.amazonaws.com;...;SSL Mode=Require
    - Kafka__ConnectionString=b-1.featbit.xxxxx.kafka.us-east-1.amazonaws.com:9092
    - Redis__ConnectionString=featbit.xxxxx.cache.amazonaws.com:6379

da-server:
  environment:
    - CLICKHOUSE_HOST=featbit.xxxxx.clickhouse.cloud
```

**Benefits**:
- Automated backups and recovery
- High availability and failover
- Auto-scaling capabilities
- Managed security and compliance
- Monitoring and alerting

## Monitoring

### Add Kafka UI

```yaml
kafka-ui:
  image: provectuslabs/kafka-ui:latest
  ports:
    - "8090:8080"
  environment:
    KAFKA_CLUSTERS_0_NAME: featbit
    KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092
    KAFKA_CLUSTERS_0_ZOOKEEPER: zookeeper:2181
  networks:
    - featbit-network
```

Access Kafka UI at http://localhost:8090

### Add Clickhouse Monitoring

```yaml
clickhouse-tabix:
  image: spoonest/clickhouse-tabix-web-client:latest
  ports:
    - "8091:80"
  networks:
    - featbit-network
```

Access ClickHouse UI at http://localhost:8091

## Performance Tuning

### Kafka Optimization

For high-throughput scenarios:

```yaml
kafka:
  environment:
    # Increase batch size
    KAFKA_BATCH_SIZE: 32768
    KAFKA_LINGER_MS: 10
    # Compression
    KAFKA_COMPRESSION_TYPE: lz4
    # Memory
    KAFKA_HEAP_OPTS: "-Xmx2G -Xms2G"
```

### ClickHouse Optimization

```yaml
clickhouse-server:
  environment:
    # Memory limits
    CLICKHOUSE_MAX_MEMORY_USAGE: 4294967296  # 4GB
    CLICKHOUSE_MAX_THREAD: 8
```

## Troubleshooting Professional Deployment

### Kafka Won't Start

```bash
# Check Zookeeper is running
docker compose ps zookeeper

# View Kafka logs
docker compose logs kafka

# Common issue: Zookeeper not ready
# Solution: Start zookeeper first, wait, then start kafka
docker compose up -d zookeeper
sleep 30
docker compose up -d kafka
```

### ClickHouse Connection Issues

```bash
# Test ClickHouse connectivity
docker compose exec clickhouse-server clickhouse-client --query "SELECT 1"

# Check ClickHouse logs
docker compose logs clickhouse-server

# Verify password
docker compose exec clickhouse-server clickhouse-client --user default --password YOUR_PASSWORD
```

### High Resource Usage

```bash
# Check resource usage
docker stats

# If memory is an issue, reduce replica count
docker compose up -d --scale evaluation-server=1

# Or allocate more resources to Docker Desktop
```

## When to Use Professional

✅ **Use Professional When**:
- Team size: >50 users
- Traffic: Millions of flag evaluations per day
- Need advanced analytics and data warehousing
- Require event streaming capabilities
- Have dedicated DevOps resources
- Budget for infrastructure costs

⚠️ **Consider Standard If**:
- Team size <50 users
- Moderate traffic
- Limited DevOps expertise
- Cost-sensitive deployment

## Reference

- **Official Documentation**: https://docs.featbit.co/installation/deployment-options
- **Source Configuration**: https://github.com/featbit/featbit/blob/main/docker-compose-pro.yml
- **Kafka Documentation**: https://kafka.apache.org/documentation/
- **ClickHouse Documentation**: https://clickhouse.com/docs/
