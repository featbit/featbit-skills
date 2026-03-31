---
name: troubleshooting-database
description: Troubleshooting guide for FeatBit database connection failures, MongoDB authentication issues, and data persistence problems.
---

# Troubleshooting: Database Issues

Database connection and persistence troubleshooting for FeatBit Docker Compose deployments.

For login/authentication failures, see [troubleshooting-login.md](troubleshooting-login.md).

## Table of Contents

1. [Database Connection Failed](#database-connection-failed)
2. [MongoDB Authentication Failed](#mongodb-authentication-failed)
3. [Data Loss After Restart](#data-loss-after-restart)

## Database Connection Failed

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

## MongoDB Authentication Failed

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

## Data Loss After Restart

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
