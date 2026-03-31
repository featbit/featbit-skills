---
name: env-vars-services
description: FeatBit service-specific environment variables for UI, Data Analytics Server, and inter-service communication, plus .env file usage for secrets management.
---

# Service Configuration Variables

## Table of Contents

1. [UI Configuration](#ui-configuration)
2. [Data Analytics Server Variables](#data-analytics-server-variables)
3. [Other Service Configuration](#other-service-configuration)
4. [Using Environment Variables with .env File](#using-environment-variables-with-env-file)

## UI Configuration

All UI environment variables must point to URLs accessible from user's browser, not Docker internal network.

### API_URL

**Description**: URL to the API server that the browser will use

**Format**: `http[s]://host[:port]`

**Examples**:
```yaml
# Development (localhost)
ui:
  environment:
    - API_URL=http://localhost:5000

# Production with domain
ui:
  environment:
    - API_URL=https://api.featbit.yourdomain.com

# Production with IP
ui:
  environment:
    - API_URL=http://192.168.1.100:5000
```

⚠️ **Important**: This URL must be reachable from the user's browser, not from inside Docker network.

### EVALUATION_URL

**Description**: URL to the Evaluation server that the browser will use

**Format**: `http[s]://host[:port]`

**Examples**:
```yaml
# Development
ui:
  environment:
    - EVALUATION_URL=http://localhost:5100

# Production
ui:
  environment:
    - EVALUATION_URL=https://eval.featbit.yourdomain.com
```

⚠️ **Important**: Must be accessible from user's browser. Supports WebSocket connections.

### DEMO_URL

**Description**: URL to demo application for trying FeatBit features

**Default**: `https://featbit-samples.vercel.app`

**Example**:
```yaml
ui:
  environment:
    - DEMO_URL=https://featbit-samples.vercel.app
    # Or use your own demo app
    - DEMO_URL=https://demo.yourdomain.com
```

### BASE_HREF

**Description**: Base path for the UI application (useful when deploying under a sub-path)

**Default**: `/`

**Examples**:
```yaml
# Root path (default)
ui:
  environment:
    - BASE_HREF=/

# Sub-path deployment
ui:
  environment:
    - BASE_HREF=/featbit/
    # Access at: https://yourdomain.com/featbit/
```

## Data Analytics Server Variables

### DB_PROVIDER

**Description**: Database provider for analytics data

**Values**: `Postgres` | `MongoDb` | `ClickHouse`

**Tier Mapping**:
- **Standalone**: `Postgres`
- **Standard**: `MongoDb` (typically)
- **Professional**: `ClickHouse`

**Examples**:
```yaml
# Standalone
da-server:
  environment:
    - DB_PROVIDER=Postgres

# Standard
da-server:
  environment:
    - DB_PROVIDER=MongoDb

# Professional
da-server:
  environment:
    - DB_PROVIDER=ClickHouse
```

### PostgreSQL Variables (when DB_PROVIDER=Postgres)

```yaml
da-server:
  environment:
    - DB_PROVIDER=Postgres
    - POSTGRES_HOST=postgresql
    - POSTGRES_PORT=5432
    - POSTGRES_USER=postgres
    - POSTGRES_PASSWORD=your_password
    - POSTGRES_DATABASE=featbit
    - CHECK_DB_LIVNESS=true  # Enable health checks
```

### MongoDB Variables (when DB_PROVIDER=MongoDb)

```yaml
da-server:
  environment:
    - DB_PROVIDER=MongoDb
    - MongoDb__ConnectionString=mongodb://admin:your_password@mongodb:27017
    - MongoDb__Database=featbit
```

### ClickHouse Variables (when DB_PROVIDER=ClickHouse)

See [env-vars-database.md](env-vars-database.md#clickhouse-data-analytics-server-only) for ClickHouse connection details.

## Other Service Configuration

### OLAP__ServiceHost

**Description**: URL to the Data Analytics server (used by api-server)

**Used by**: `api-server`

**Format**: `http://host[:port]`

**Example**:
```yaml
api-server:
  environment:
    - OLAP__ServiceHost=http://da-server
    # Or with custom port
    - OLAP__ServiceHost=http://da-server:8200
```

## Using Environment Variables with .env File

Create a `.env` file to manage secrets:

```bash
# .env file
DB_PASSWORD=strong_postgres_password
MONGO_PASSWORD=strong_mongo_password
REDIS_PASSWORD=strong_redis_password
CLICKHOUSE_PASSWORD=strong_clickhouse_password
```

Reference in `docker-compose.yml`:

```yaml
services:
  api-server:
    env_file: .env
    environment:
      - Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password=${DB_PASSWORD};Database=featbit
      - Redis__ConnectionString=redis:6379,password=${REDIS_PASSWORD}
```

⚠️ **Security**: Add `.env` to `.gitignore` to avoid committing secrets.
