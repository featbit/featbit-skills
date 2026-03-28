---
name: troubleshooting-setup
description: Troubleshooting setup issues for FeatBit Docker deployments including login, ports, UI connectivity, database, and service startup
---

# Troubleshooting: Setup Issues

## Table of Contents

1. [Login Failed / Cannot Sign In](#login-failed--cannot-sign-in)
2. [Port Already in Use](#port-already-in-use)
3. [UI Cannot Connect to API](#ui-cannot-connect-to-api)
4. [Database Connection Failed](#database-connection-failed)
5. [Service Won't Start](#service-wont-start)

## Common Issues

### Login Failed / Cannot Sign In

**Symptom**: Unable to log in with default credentials (test@featbit.com / 123456), login page shows error or keeps reloading

---

#### Understanding the Problem

When you cannot log in to FeatBit, there are **two main causes**:

**1. Database Not Initialized**
- The database exists but is **empty** (no tables, collections, or seed data)
- This happens when database initialization scripts didn't run automatically
- Without seed data, there are no users to authenticate

**2. API Server Not Running or Failing**
- The API server handles authentication requests
- If it's not running or crashing, login requests fail
- Check if API server is actually started and healthy

---

#### Solution 1: Fix Database Initialization

**Understanding**: FeatBit requires database schemas and seed data (including the default test user). If Docker Compose didn't run initialization scripts automatically, you need to run them manually.

**⚠️ Important**: Scripts must be executed **in version order** (v0.0.0 → v1.0.0 → v1.1.0...) because later versions depend on earlier ones.

##### Step 1: Check if Database Has Data

**PostgreSQL**:
```bash
docker compose exec postgresql psql -U postgres -d featbit -c "\dt"
```
If output shows "Did not find any relations", database is empty.

**MongoDB**:
```bash
docker compose exec mongodb mongosh featbit --eval "db.getCollectionNames()"
```
If output shows empty array `[]`, database is empty.

##### Step 2: Run Initialization Scripts

**For PostgreSQL**:

Navigate to your cloned FeatBit repository first:
```bash
cd path/to/featbit  # Navigate to where you cloned the repository
```

List available script versions:
```bash
# Windows PowerShell:
dir infra\postgresql\docker-entrypoint-initdb.d\

# Linux/Mac:
ls infra/postgresql/docker-entrypoint-initdb.d/
```

Execute scripts in version order:

**Windows PowerShell**:
```powershell
# v0.0.0 (initial schema and seed data)
Get-Content infra\postgresql\docker-entrypoint-initdb.d\v0.0.0.sql | docker compose exec -T postgresql psql -U postgres -d featbit

# v1.0.0 (if exists)
Get-Content infra\postgresql\docker-entrypoint-initdb.d\v1.0.0.sql | docker compose exec -T postgresql psql -U postgres -d featbit

# Continue for each version file you see...
```

**Linux/Mac**:
```bash
# v0.0.0 (initial schema and seed data)
docker compose exec -T postgresql psql -U postgres -d featbit < infra/postgresql/docker-entrypoint-initdb.d/v0.0.0.sql

# v1.0.0 (if exists)
docker compose exec -T postgresql psql -U postgres -d featbit < infra/postgresql/docker-entrypoint-initdb.d/v1.0.0.sql

# Continue for each version file you see...
```

**For MongoDB**:

Navigate to your cloned FeatBit repository first:
```bash
cd path/to/featbit  # Navigate to where you cloned the repository
```

List available script versions:
```bash
# Windows PowerShell:
dir infra\mongodb\docker-entrypoint-initdb.d\

# Linux/Mac:
ls infra/mongodb/docker-entrypoint-initdb.d/
```

Execute scripts in version order:

**Windows PowerShell**:
```powershell
# v0.0.0 (initial collections and seed data)
Get-Content infra\mongodb\docker-entrypoint-initdb.d\v0.0.0.js | docker compose exec -T mongodb mongosh featbit

# v1.0.0 (if exists)
Get-Content infra\mongodb\docker-entrypoint-initdb.d\v1.0.0.js | docker compose exec -T mongodb mongosh featbit

# Continue for each version file you see...
```

**Linux/Mac**:
```bash
# v0.0.0 (initial collections and seed data)
docker compose exec -T mongodb mongosh featbit < infra/mongodb/docker-entrypoint-initdb.d/v0.0.0.js

# v1.0.0 (if exists)
docker compose exec -T mongodb mongosh featbit < infra/mongodb/docker-entrypoint-initdb.d/v1.0.0.js

# Continue for each version file you see...
```

**Script Locations**:
- PostgreSQL: https://github.com/featbit/featbit/tree/main/infra/postgresql/docker-entrypoint-initdb.d
- MongoDB: https://github.com/featbit/featbit/tree/main/infra/mongodb/docker-entrypoint-initdb.d

##### Step 3: Verify Initialization

**PostgreSQL**:
```bash
# Should now show multiple tables
docker compose exec postgresql psql -U postgres -d featbit -c "\dt"

# Check if user exists
docker compose exec postgresql psql -U postgres -d featbit -c "SELECT email FROM \"Account\" WHERE email='test@featbit.com';"
```

**MongoDB**:
```bash
# Should show collections
docker compose exec mongodb mongosh featbit --eval "db.getCollectionNames()"

# Check if user exists
docker compose exec mongodb mongosh featbit --eval "db.Accounts.findOne({email: 'test@featbit.com'})"
```

---

#### Solution 2: Fix API Server Issues

**Understanding**: The API server must be running and healthy to process login requests. If it's crashing or not started, you'll see login failures even if the database is fine.

##### Step 1: Check API Server Status

```bash
docker compose ps api-server
```

**Expected**: Status should be "Up" (running)
**Problem**: Status shows "Exit", "Restarting", or service is missing

##### Step 2: Check API Server Logs

```bash
# View recent logs
docker compose logs --tail=50 api-server

# Follow live logs (press Ctrl+C to stop)
docker compose logs -f api-server
```

**Look for errors** like:
- "Failed to connect to database" → Database connection issue
- "Connection refused" → Database not accessible
- "Environment variable 'DbProvider' is required" → Configuration missing
- Port conflict errors → Port 5000 already in use

##### Step 3: Common Fixes

**If API server crashed or exited**:
```bash
# Restart the API server
docker compose restart api-server

# Watch logs to see if it starts successfully
docker compose logs -f api-server
```

**If database connection errors**:
```bash
# Verify connection string configuration
docker compose config | grep -i connectionstring

# Check if database is running
docker compose ps postgresql  # or mongodb
```

**If environment configuration errors**:
```bash
# Check environment variables are set
docker compose config | grep -A 20 api-server

# Look for DbProvider, ConnectionString, etc.
```

**If port conflict (port 5000 already in use)**:

Windows PowerShell:
```powershell
# Find what's using port 5000
netstat -ano | findstr :5000
```

Linux/Mac:
```bash
lsof -i :5000
```

**Force recreate API server** (if other fixes don't work):
```bash
docker compose up -d --force-recreate api-server
docker compose logs -f api-server
```

##### Step 4: Test API Health

After API server is running:

```bash
# Should return "Healthy" or 200 OK
curl http://localhost:5000/health/liveness
```

**Windows PowerShell** (if curl not available):
```powershell
Invoke-WebRequest -Uri http://localhost:5000/health/liveness
```

**If health check fails**, API server is not fully started. Check logs again.

---

#### Quick Checklist

Use this checklist to diagnose login issues:

- [ ] Database container is running: `docker compose ps postgresql` or `docker compose ps mongodb`
- [ ] Database has data: Run check commands from Solution 1 Step 1
- [ ] API server container is running: `docker compose ps api-server`
- [ ] API server has no errors: `docker compose logs --tail=50 api-server`
- [ ] API server is healthy: `curl http://localhost:5000/health/liveness`
- [ ] UI can reach API: Check browser console for network errors (F12)

---

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
