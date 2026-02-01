# Troubleshooting Guide

Comprehensive troubleshooting guide for FeatBit Docker Compose deployments.

## Common Issues

### Port Already in Use

**Symptom**: Error when starting: "port is already allocated"

**Diagnosis**:
```bash
# Windows
netstat -ano | findstr :8081
netstat -ano | findstr :5000

# Linux/Mac
lsof -i :8081
lsof -i :5000
ss -tulpn | grep :8081
```

**Solutions**:

1. **Stop conflicting service**:
```bash
# Find and stop the process using the port
# Windows: taskkill /PID <PID> /F
# Linux/Mac: kill -9 <PID>
```

2. **Change port mapping**:
```yaml
ui:
  ports:
    - "9081:80"  # Changed from 8081 to 9081

api-server:
  ports:
    - "6000:5000"  # Changed from 5000 to 6000
```

Don't forget to update UI environment variables:
```yaml
ui:
  environment:
    - API_URL=http://localhost:6000  # Updated port
```

### UI Cannot Connect to API

**Symptom**: UI loads but shows "Cannot connect to API" or "Network Error"

**Root Cause**: `API_URL` not accessible from browser

**Diagnosis**:
```bash
# Test from your machine (NOT from inside Docker)
curl http://localhost:5000/health/liveness

# If using domain
curl https://api.featbit.yourdomain.com/health/liveness
```

**Solutions**:

1. **Check API_URL configuration**:
```yaml
ui:
  environment:
    # ❌ Wrong: Docker internal hostname
    - API_URL=http://api-server:5000
    
    # ✅ Correct: URL accessible from browser
    - API_URL=http://localhost:5000
    # Or for production:
    - API_URL=https://api.featbit.yourdomain.com
```

2. **Check API server is running**:
```bash
docker compose ps api-server
docker compose logs api-server
```

3. **Check CORS settings** (if using different domains):
API server should allow requests from UI origin.

### Database Connection Failed

**Symptom**: Service fails to start with "connection refused" or "authentication failed"

**Diagnosis**:
```bash
# Check database is running
docker compose ps postgresql
docker compose ps mongodb

# View database logs
docker compose logs postgresql
docker compose logs mongodb

# Test PostgreSQL connection
docker compose exec postgresql pg_isready -U postgres

# Test MongoDB connection
docker compose exec mongodb mongosh --eval "db.adminCommand('ping')"
```

**Common Causes & Solutions**:

1. **Database not ready yet**:
```yaml
api-server:
  depends_on:
    postgresql:
      condition: service_healthy  # Wait for health check

postgresql:
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U postgres"]
    interval: 5s
    timeout: 5s
    retries: 5
```

2. **Wrong password**:
```yaml
# Ensure passwords match
postgresql:
  environment:
    POSTGRES_PASSWORD: your_password

api-server:
  environment:
    - Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password=your_password;Database=featbit
```

3. **Typo in connection string**:
```yaml
# Check carefully for typos
- Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password=your_password;Database=featbit
#                            ^^^^         ^^^^           ^^^^
#                            hostname     port           password
```

4. **Network issue**:
```bash
# Ensure all services are on same network
docker compose exec api-server ping postgresql
```

### Service Won't Start

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

## Emergency Recovery

### Complete Reset

If all else fails, completely reset the deployment:

```bash
# ⚠️ WARNING: This will delete ALL data!

# Stop all services
docker compose down

# Remove all volumes (data will be lost!)
docker compose down -v

# Remove all containers
docker rm -f $(docker ps -aq)

# Clean up Docker
docker system prune -a --volumes

# Start fresh
docker compose pull
docker compose up -d
```

### Restore from Backup

**PostgreSQL**:
```bash
# Stop services
docker compose down

# Remove old volume
docker volume rm featbit_postgres

# Start PostgreSQL
docker compose up -d postgresql

# Wait for PostgreSQL to be ready
sleep 10

# Restore backup
cat backup_20260201.sql | docker compose exec -T postgresql psql -U postgres featbit

# Start other services
docker compose up -d
```

**MongoDB**:
```bash
# Stop services
docker compose down

# Remove old volume
docker volume rm featbit_mongodb

# Start MongoDB
docker compose up -d mongodb

# Wait for MongoDB to be ready
sleep 10

# Restore backup
docker compose exec mongodb mongorestore /backup/featbit --db featbit

# Start other services
docker compose up -d
```

## Debugging Tips

### Enable Debug Logging

```yaml
api-server:
  environment:
    - Logging__LogLevel__Default=Debug
    - Logging__LogLevel__Microsoft=Information
```

### Access Container Shell

```bash
# Access container shell for debugging
docker compose exec api-server /bin/bash
docker compose exec postgresql /bin/bash

# Run commands inside container
docker compose exec api-server env  # View environment variables
docker compose exec api-server ps aux  # View processes
```

### Network Debugging

```bash
# Check network connectivity between containers
docker compose exec api-server ping postgresql
docker compose exec api-server nslookup postgresql
docker compose exec api-server curl http://redis:6379

# Inspect network
docker network inspect featbit_featbit-network
```

### View Real-Time Logs

```bash
# Follow logs for all services
docker compose logs -f

# Follow logs for specific service
docker compose logs -f api-server

# Last 100 lines
docker compose logs --tail=100 api-server

# Logs since specific time
docker compose logs --since 2024-02-01T10:00:00 api-server
```

## Getting Help

If you can't resolve the issue:

1. **Check official documentation**: https://docs.featbit.co
2. **Search GitHub issues**: https://github.com/featbit/featbit/issues
3. **Join Slack community**: https://join.slack.com/t/featbit/shared_invite/zt-1ew5e2vbb-x6Apan1xZOaYMnFzqZkGNQ
4. **Create GitHub issue** with:
   - FeatBit version
   - Docker Compose version
   - Deployment tier (Standalone/Standard/Professional)
   - Full docker-compose.yml (remove passwords!)
   - Error logs
   - Steps to reproduce

## Reference

- **Official Documentation**: https://docs.featbit.co/installation/docker-compose
- **GitHub Issues**: https://github.com/featbit/featbit/issues
- **FAQ**: https://docs.featbit.co/installation/faq
