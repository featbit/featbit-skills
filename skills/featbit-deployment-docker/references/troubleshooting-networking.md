---
name: troubleshooting-networking
description: Troubleshooting guide for FeatBit networking issues including port conflicts, UI-API connectivity, WebSocket connections, Redis connectivity, and container health checks.
---

# Troubleshooting: Networking Issues

Network and connectivity troubleshooting for FeatBit Docker Compose deployments.

## Table of Contents

1. [Port Already in Use](#port-already-in-use)
2. [UI Cannot Connect to API](#ui-cannot-connect-to-api)
3. [Evaluation Server WebSocket Connection Failed](#evaluation-server-websocket-connection-failed)
4. [Redis Connection Issues](#redis-connection-issues)
5. [Container Healthy but Not Responding](#container-healthy-but-not-responding)

## Port Already in Use

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

## UI Cannot Connect to API

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
    # Wrong: Docker internal hostname
    - API_URL=http://api-server:5000

    # Correct: URL accessible from browser
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

## Evaluation Server WebSocket Connection Failed

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

## Redis Connection Issues

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

## Container Healthy but Not Responding

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
