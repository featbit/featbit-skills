# Standalone Deployment - Complete Configuration

Complete docker-compose.yml and detailed setup guide for FeatBit Standalone deployment.

## Complete docker-compose.yml

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
      - MqProvider=Postgres
      - CacheProvider=None
      - Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password=please_change_me;Database=featbit
    depends_on:
      - postgresql
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

volumes:
  postgres:

networks:
  featbit-network:
    name: featbit-network
    driver: bridge
```

## Deployment Steps

### Step 1: Download Configuration

```bash
# Option 1: Download from official repository
curl -O https://raw.githubusercontent.com/featbit/featbit/main/docker-compose.yml

# Option 2: Create file manually
# Copy the configuration above to docker-compose.yml
```

### Step 2: Customize Configuration

**Security (Required)**:
```bash
# Change default PostgreSQL password
# Edit docker-compose.yml and replace 'please_change_me' with strong password
```

**Networking (Optional)**:
```yaml
# If deploying to production, update UI environment variables
ui:
  environment:
    - API_URL=https://api.yourdomain.com       # Public API URL
    - EVALUATION_URL=https://eval.yourdomain.com  # Public Evaluation URL
```

### Step 3: Deploy

```bash
# Start all services
docker compose up -d

# Verify all services are running
docker compose ps

# Expected output:
# NAME                IMAGE                                        STATUS
# ui                  featbit/featbit-ui:latest                   Up
# api-server          featbit/featbit-api-server:latest           Up (healthy)
# evaluation-server   featbit/featbit-evaluation-server:latest    Up (healthy)
# da-server           featbit/featbit-data-analytics-server:latest Up
# postgresql          postgres:15.10                               Up (healthy)
```

### Step 4: Initial Access

1. Open browser: http://localhost:8081
2. Login with default credentials:
   - Email: `test@featbit.com`
   - Password: `123456`
3. **Immediately change password** after first login

## Resource Requirements

**Minimum**:
- RAM: 2GB
- CPU: 1 core
- Disk: 10GB

**Recommended**:
- RAM: 4GB
- CPU: 2 cores
- Disk: 20GB

## Port Usage

| Service | Port | Description |
|---------|------|-------------|
| UI | 8081 | Web interface (browser access) |
| API Server | 5000 | API endpoints (browser access required) |
| Evaluation Server | 5100 | Flag evaluation (browser access required) |
| Data Analytics | 8200 | Internal analytics service |
| PostgreSQL | 5432 | Database (internal use only) |

## Using with Managed PostgreSQL

Replace embedded PostgreSQL with AWS RDS, Azure Database, or other managed services:

```yaml
api-server:
  environment:
    # AWS RDS example
    - Postgres__ConnectionString=Host=featbit-db.xxxxx.us-east-1.rds.amazonaws.com;Port=5432;Username=admin;Password=${DB_PASSWORD};Database=featbit;SSL Mode=Require

evaluation-server:
  environment:
    # Same connection string as api-server
    - Postgres__ConnectionString=Host=featbit-db.xxxxx.us-east-1.rds.amazonaws.com;Port=5432;Username=admin;Password=${DB_PASSWORD};Database=featbit;SSL Mode=Require

da-server:
  environment:
    POSTGRES_HOST: featbit-db.xxxxx.us-east-1.rds.amazonaws.com
    POSTGRES_PORT: 5432
    POSTGRES_USER: admin
    POSTGRES_PASSWORD: ${DB_PASSWORD}
    POSTGRES_DATABASE: featbit

# Remove or comment out the postgresql service
# postgresql:
#   ...
```

**Benefits of Managed Database**:
- Automated backups
- High availability
- Automated maintenance and updates
- Better security and compliance
- Scalability

## Reference

- **Official Documentation**: https://docs.featbit.co/installation/docker-compose
- **Source Configuration**: https://github.com/featbit/featbit/blob/main/docker-compose.yml
- **PostgreSQL Init Scripts**: https://github.com/featbit/featbit/tree/main/infra/postgresql/docker-entrypoint-initdb.d
