---
name: troubleshooting-login
description: Troubleshooting guide for FeatBit login and authentication failures including database initialization and API server issues.
---

# Troubleshooting: Login & Authentication

Login and authentication troubleshooting for FeatBit Docker Compose deployments.

## Table of Contents

1. [Understanding the Problem](#understanding-the-problem)
2. [Solution 1: Fix Database Initialization](#solution-1-fix-database-initialization)
3. [Solution 2: Fix API Server Issues](#solution-2-fix-api-server-issues)
4. [Quick Checklist](#quick-checklist)

## Login Failed / Cannot Sign In

**Symptom**: Unable to log in with default credentials (test@featbit.com / 123456), login page shows error or keeps reloading

---

### Understanding the Problem

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

### Solution 1: Fix Database Initialization

**Understanding**: FeatBit requires database schemas and seed data (including the default test user). If Docker Compose didn't run initialization scripts automatically, you need to run them manually.

**Important**: Scripts must be executed **in version order** (v0.0.0 → v1.0.0 → v1.1.0...) because later versions depend on earlier ones.

#### Step 1: Check if Database Has Data

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

#### Step 2: Run Initialization Scripts

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

#### Step 3: Verify Initialization

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

### Solution 2: Fix API Server Issues

**Understanding**: The API server must be running and healthy to process login requests. If it's crashing or not started, you'll see login failures even if the database is fine.

#### Step 1: Check API Server Status

```bash
docker compose ps api-server
```

**Expected**: Status should be "Up" (running)
**Problem**: Status shows "Exit", "Restarting", or service is missing

#### Step 2: Check API Server Logs

```bash
# View recent logs
docker compose logs --tail=50 api-server

# Follow live logs (press Ctrl+C to stop)
docker compose logs -f api-server
```

**Look for errors** like:
- "Failed to connect to database" -> Database connection issue
- "Connection refused" -> Database not accessible
- "Environment variable 'DbProvider' is required" -> Configuration missing
- Port conflict errors -> Port 5000 already in use

#### Step 3: Common Fixes

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

#### Step 4: Test API Health

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

### Quick Checklist

Use this checklist to diagnose login issues:

- [ ] Database container is running: `docker compose ps postgresql` or `docker compose ps mongodb`
- [ ] Database has data: Run check commands from Solution 1 Step 1
- [ ] API server container is running: `docker compose ps api-server`
- [ ] API server has no errors: `docker compose logs --tail=50 api-server`
- [ ] API server is healthy: `curl http://localhost:5000/health/liveness`
- [ ] UI can reach API: Check browser console for network errors (F12)
