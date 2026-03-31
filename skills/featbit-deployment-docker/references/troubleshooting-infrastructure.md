---
name: troubleshooting-infrastructure
description: Troubleshooting guide for FeatBit infrastructure issues including service startup failures, Kafka, ClickHouse, disk space, and performance problems.
---

# Troubleshooting: Infrastructure Issues

Infrastructure and service-level troubleshooting for FeatBit Docker Compose deployments.

## Table of Contents

1. [Service Won't Start](#service-wont-start)
2. [Kafka Won't Start](#kafka-wont-start)
3. [ClickHouse Connection Failed](#clickhouse-connection-failed)
4. [Disk Space Issues](#disk-space-issues)
5. [Performance Issues](#performance-issues)

## Service Won't Start

**Symptom**: Service shows "Exit 1" or constantly restarting

**Diagnosis**:
```bash
# Check service status
docker compose ps

# View logs for specific service
docker compose logs api-server
docker compose logs -f api-server  # Follow logs

# Check previous container logs
docker compose logs --tail=100 api-server
```

**Common Causes**:

1. **Missing environment variables**:
```bash
# Check logs for error like:
# "Environment variable 'DbProvider' is required"

# Solution: Add missing variable
api-server:
  environment:
    - DbProvider=Postgres  # Add this
```

2. **Invalid configuration**:
```bash
# Look for validation errors in logs
# Fix configuration based on error message
```

3. **Resource constraints**:
```bash
# Check Docker resources
docker stats

# Increase Docker Desktop memory/CPU if needed
# Or reduce replicas/resource limits
```

4. **Image pull failure**:
```bash
# Pull images manually
docker compose pull

# Check internet connectivity
# Check Docker Hub status
```

## Kafka Won't Start

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

## ClickHouse Connection Failed

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

## Disk Space Issues

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

## Performance Issues

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
