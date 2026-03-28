---
name: troubleshooting-infrastructure
description: Troubleshooting infrastructure issues for FeatBit Docker deployments including WebSocket, Redis, MongoDB, Kafka, ClickHouse, disk, and performance
---

# Troubleshooting: Infrastructure Issues

## Table of Contents

1. [Evaluation Server WebSocket Connection Failed](#evaluation-server-websocket-connection-failed)
2. [Redis Connection Issues](#redis-connection-issues)
3. [MongoDB Authentication Failed](#mongodb-authentication-failed)
4. [Kafka Won't Start](#kafka-wont-start)
5. [ClickHouse Connection Failed](#clickhouse-connection-failed)
6. [Container Healthy but Not Responding](#container-healthy-but-not-responding)
7. [Disk Space Issues](#disk-space-issues)
8. [Performance Issues](#performance-issues)
9. [Data Loss After Restart](#data-loss-after-restart)

### Evaluation Server WebSocket Connection Failed

**Symptom**: UI can't establish WebSocket connection to evaluation server

**Diagnosis**:
```bash
# Test WebSocket endpoint (from browser console)
ws://localhost:5100

# Check evaluation server logs
docker compose logs evaluation-server
```

**Solutions**:

1. **Check EVALUATION_URL**:
```yaml
ui:
  environment:
    # Must be accessible from browser
    - EVALUATION_URL=http://localhost:5100
```

2. **Check reverse proxy WebSocket support**:
```nginx
# nginx configuration
location / {
    proxy_pass http://evaluation-server:5100;

    # WebSocket support required
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_read_timeout 86400;
}
```

### Redis Connection Issues

**Symptom**: "Redis connection failed" or "ECONNREFUSED redis:6379"

**Diagnosis**:
```bash
# Check Redis is running
docker compose ps redis

# Test Redis connection
docker compose exec redis redis-cli ping
# Should return: PONG

# Check Redis logs
docker compose logs redis
```

**Solutions**:

1. **Redis not started**:
```bash
docker compose up -d redis
```

2. **Wrong Redis connection string**:
```yaml
api-server:
  environment:
    # No password
    - Redis__ConnectionString=redis:6379

    # With password
    - Redis__ConnectionString=redis:6379,password=your_password
```

3. **Redis out of memory**:
```bash
# Check Redis memory
docker compose exec redis redis-cli INFO memory

# Increase maxmemory
redis:
  command: redis-server --maxmemory 1gb
```

### MongoDB Authentication Failed

**Symptom**: "Authentication failed" connecting to MongoDB

**Common Issues**:

1. **Missing initialization script**:
```bash
# Download init script
mkdir -p infra/mongodb/docker-entrypoint-initdb.d
curl -o infra/mongodb/docker-entrypoint-initdb.d/v0.0.0.js \
  https://raw.githubusercontent.com/featbit/featbit/main/infra/mongodb/docker-entrypoint-initdb.d/v0.0.0.js

# Recreate MongoDB with init script
docker compose stop mongodb
docker volume rm featbit_mongodb
docker compose up -d mongodb
```

2. **Wrong credentials**:
```yaml
mongodb:
  environment:
    MONGO_INITDB_ROOT_USERNAME: admin
    MONGO_INITDB_ROOT_PASSWORD: your_password

api-server:
  environment:
    # Match MongoDB credentials
    - MongoDb__ConnectionString=mongodb://admin:your_password@mongodb:27017
```

### Kafka Won't Start

**Symptom**: Kafka service constantly restarting (Professional tier)

**Common Causes**:

1. **Zookeeper not ready**:
```bash
# Start Zookeeper first
docker compose up -d zookeeper

# Wait 30 seconds
sleep 30

# Then start Kafka
docker compose up -d kafka
```

2. **Port conflict**:
```bash
# Check if port 9092 is in use
netstat -ano | findstr :9092  # Windows
lsof -i :9092  # Linux/Mac
```

3. **Insufficient resources**:
```bash
# Kafka requires significant memory
# Increase Docker memory to at least 8GB
```

### ClickHouse Connection Failed

**Symptom**: Data Analytics server can't connect to ClickHouse

**Diagnosis**:
```bash
# Test ClickHouse connection
docker compose exec clickhouse-server clickhouse-client --query "SELECT 1"

# Check ClickHouse logs
docker compose logs clickhouse-server
```

**Solutions**:

1. **ClickHouse not ready**:
```bash
# ClickHouse takes longer to start
# Wait 60 seconds after starting
```

2. **Wrong credentials**:
```yaml
clickhouse-server:
  environment:
    CLICKHOUSE_PASSWORD: your_password

da-server:
  environment:
    - CLICKHOUSE_PASSWORD=your_password  # Must match
```

### Container Healthy but Not Responding

**Symptom**: Health check passes but service doesn't respond

**Diagnosis**:
```bash
# Check if service is actually listening
docker compose exec api-server netstat -tulpn | grep 5000

# Test directly from inside container
docker compose exec api-server curl http://localhost:5000/health/liveness

# Check from host
curl http://localhost:5000/health/liveness
```

**Solutions**:

1. **Port mapping issue**:
```yaml
api-server:
  ports:
    - "5000:5000"  # host:container
    #   ^^^^  ^^^^
    #   Host  Container port
```

2. **Service binding to wrong interface**:
Service should bind to `0.0.0.0` not `127.0.0.1`

### Disk Space Issues

**Symptom**: "No space left on device" errors

**Diagnosis**:
```bash
# Check Docker disk usage
docker system df

# Check available disk space
df -h
```

**Solutions**:

1. **Clean up Docker**:
```bash
# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove everything unused
docker system prune -a --volumes
```

2. **Limit log file sizes**:
```yaml
api-server:
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "3"
```

3. **Move Docker data directory**:
```bash
# Change Docker data directory to larger disk
# Docker Desktop: Settings > Resources > Advanced > Disk image location
```

### Performance Issues

**Symptom**: Slow response times, high latency

**Diagnosis**:
```bash
# Check resource usage
docker stats

# Check service logs for errors
docker compose logs api-server | grep -i error
docker compose logs evaluation-server | grep -i error
```

**Solutions**:

1. **Insufficient resources**:
```bash
# Increase Docker Desktop resources
# Settings > Resources > Advanced
# - Memory: At least 4GB (8GB+ for Professional)
# - CPUs: At least 2 (4+ for Professional)
```

2. **Database performance**:
```bash
# Check database slow queries
docker compose exec postgresql psql -U postgres -d featbit -c "SELECT * FROM pg_stat_activity WHERE state = 'active';"
```

3. **Enable caching** (if using Standalone):
Upgrade to Standard tier with Redis caching

4. **Check network latency**:
If using remote databases, latency may be high

### Data Loss After Restart

**Symptom**: Data disappears after stopping and restarting containers

**Cause**: Not using volumes for data persistence

**Solution**:
```yaml
postgresql:
  volumes:
    - postgres:/var/lib/postgresql/data  # Persist data

volumes:
  postgres:  # Define named volume
```

**Verify volumes exist**:
```bash
docker volume ls | grep postgres
```
