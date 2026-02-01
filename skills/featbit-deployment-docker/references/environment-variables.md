# Environment Variables Reference

Complete reference for all FeatBit Docker Compose environment variables.

## Provider Configuration

These variables determine which infrastructure components FeatBit services use.

### DbProvider

Controls the database type for data storage.

**Values**: `Postgres` | `MongoDb`

**Used by**: `api-server`, `evaluation-server`, `da-server`

**Examples**:
```yaml
# PostgreSQL
api-server:
  environment:
    - DbProvider=Postgres

# MongoDB
api-server:
  environment:
    - DbProvider=MongoDb
```

### MqProvider

Controls the message queue provider for inter-service communication.

**Values**: `Postgres` | `Redis` | `Kafka`

**Used by**: `api-server`, `evaluation-server`

**Tier Mapping**:
- **Standalone**: `Postgres`
- **Standard**: `Redis`
- **Professional**: `Kafka`

**Examples**:
```yaml
# Standalone: PostgreSQL as message queue
api-server:
  environment:
    - MqProvider=Postgres

# Standard: Redis as message queue
api-server:
  environment:
    - MqProvider=Redis

# Professional: Kafka as message queue
api-server:
  environment:
    - MqProvider=Kafka
```

### CacheProvider

Controls the caching layer for improved performance.

**Values**: `None` | `Redis`

**Used by**: `api-server`, `evaluation-server`

**Tier Mapping**:
- **Standalone**: `None`
- **Standard**: `Redis`
- **Professional**: `Redis`

**Examples**:
```yaml
# No caching (Standalone)
api-server:
  environment:
    - CacheProvider=None

# Redis caching (Standard/Professional)
api-server:
  environment:
    - CacheProvider=Redis
```

## Database Connection Strings

### PostgreSQL

**Variable**: `Postgres__ConnectionString`

**Format**: `Host={host};Port={port};Username={user};Password={password};Database={database}[;SSL Mode={mode}]`

**Examples**:
```yaml
# Local PostgreSQL
Postgres__ConnectionString: Host=postgresql;Port=5432;Username=postgres;Password=your_password;Database=featbit

# AWS RDS with SSL
Postgres__ConnectionString: Host=featbit.xxxxx.rds.amazonaws.com;Port=5432;Username=admin;Password=your_password;Database=featbit;SSL Mode=Require

# Azure Database for PostgreSQL
Postgres__ConnectionString: Host=featbit.postgres.database.azure.com;Port=5432;Username=admin@featbit;Password=your_password;Database=featbit;SSL Mode=Require
```

**Common SSL Modes**:
- `Disable`: No SSL
- `Require`: Require SSL
- `Prefer`: Use SSL if available

### MongoDB

**Variable**: `MongoDb__ConnectionString`

**Format**: `mongodb://[username:password@]host[:port][/[defaultauthdb]][?options]`

**Variable**: `MongoDb__Database`

**Examples**:
```yaml
# Local MongoDB with authentication
MongoDb__ConnectionString: mongodb://admin:your_password@mongodb:27017
MongoDb__Database: featbit

# MongoDB Atlas
MongoDb__ConnectionString: mongodb+srv://admin:your_password@cluster.xxxxx.mongodb.net
MongoDb__Database: featbit

# Local MongoDB without authentication (development only)
MongoDb__ConnectionString: mongodb://mongodb:27017
MongoDb__Database: featbit
```

### Redis

**Variable**: `Redis__ConnectionString`

**Format**: `{host}:{port}[,password={password}][,ssl={true|false}]`

**Examples**:
```yaml
# Local Redis (no password)
Redis__ConnectionString: redis:6379

# Redis with password
Redis__ConnectionString: redis:6379,password=your_password

# AWS ElastiCache
Redis__ConnectionString: featbit.xxxxx.cache.amazonaws.com:6379

# Azure Cache for Redis with SSL
Redis__ConnectionString: featbit.redis.cache.windows.net:6380,password=your_password,ssl=True
```

### Kafka

**Variable**: `Kafka__ConnectionString`

**Format**: `{host}:{port}[,{host2}:{port2},...]`

**Examples**:
```yaml
# Local Kafka
Kafka__ConnectionString: kafka:9092

# Kafka cluster (multiple brokers)
Kafka__ConnectionString: kafka1:9092,kafka2:9092,kafka3:9092

# AWS MSK
Kafka__ConnectionString: b-1.featbit.xxxxx.kafka.us-east-1.amazonaws.com:9092,b-2.featbit.xxxxx.kafka.us-east-1.amazonaws.com:9092
```

### ClickHouse (Data Analytics Server only)

**Variables**:
- `CLICKHOUSE_HOST`: ClickHouse server hostname
- `CLICKHOUSE_PORT`: ClickHouse HTTP port (typically 8123)
- `CLICKHOUSE_USER`: Database user
- `CLICKHOUSE_PASSWORD`: Database password
- `CLICKHOUSE_DATABASE`: Database name

**Examples**:
```yaml
# Local ClickHouse
da-server:
  environment:
    - DB_PROVIDER=ClickHouse
    - CLICKHOUSE_HOST=clickhouse-server
    - CLICKHOUSE_PORT=8123
    - CLICKHOUSE_USER=default
    - CLICKHOUSE_PASSWORD=your_password
    - CLICKHOUSE_DATABASE=featbit

# ClickHouse Cloud
da-server:
  environment:
    - DB_PROVIDER=ClickHouse
    - CLICKHOUSE_HOST=featbit.clickhouse.cloud
    - CLICKHOUSE_PORT=8443
    - CLICKHOUSE_USER=default
    - CLICKHOUSE_PASSWORD=your_password
    - CLICKHOUSE_DATABASE=featbit
```

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

See ClickHouse section above.

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

## Environment-Specific Configurations

### Development

```yaml
api-server:
  environment:
    - DbProvider=Postgres
    - MqProvider=Postgres
    - CacheProvider=None
    - Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password=dev_password;Database=featbit
```

### Staging

```yaml
api-server:
  environment:
    - DbProvider=Postgres
    - MqProvider=Redis
    - CacheProvider=Redis
    - Postgres__ConnectionString=Host=staging-db.xxxxx.rds.amazonaws.com;Port=5432;Username=admin;Password=${DB_PASSWORD};Database=featbit;SSL Mode=Require
    - Redis__ConnectionString=staging-redis.xxxxx.cache.amazonaws.com:6379,password=${REDIS_PASSWORD}
```

### Production

```yaml
api-server:
  environment:
    - DbProvider=Postgres
    - MqProvider=Kafka
    - CacheProvider=Redis
    - Postgres__ConnectionString=Host=prod-db.xxxxx.rds.amazonaws.com;Port=5432;Username=admin;Password=${DB_PASSWORD};Database=featbit;SSL Mode=Require
    - Kafka__ConnectionString=b-1.prod-kafka.xxxxx.kafka.us-east-1.amazonaws.com:9092,b-2.prod-kafka.xxxxx.kafka.us-east-1.amazonaws.com:9092
    - Redis__ConnectionString=prod-redis.xxxxx.cache.amazonaws.com:6379,password=${REDIS_PASSWORD}
```

## Quick Reference Table

| Variable | Service | Standalone | Standard | Professional |
|----------|---------|------------|----------|--------------|
| `DbProvider` | api-server, evaluation-server | `Postgres` | `Postgres` or `MongoDb` | `Postgres` or `MongoDb` |
| `MqProvider` | api-server, evaluation-server | `Postgres` | `Redis` | `Kafka` |
| `CacheProvider` | api-server, evaluation-server | `None` | `Redis` | `Redis` |
| `DB_PROVIDER` | da-server | `Postgres` | `MongoDb` | `ClickHouse` |
| `API_URL` | ui | Browser-accessible API URL | Browser-accessible API URL | Browser-accessible API URL |
| `EVALUATION_URL` | ui | Browser-accessible Eval URL | Browser-accessible Eval URL | Browser-accessible Eval URL |

## Troubleshooting Environment Variables

### UI Cannot Connect to API

**Problem**: UI shows "Cannot connect to API" error

**Solution**: Verify `API_URL` is accessible from browser:
```bash
# Test from your machine (not inside Docker)
curl http://localhost:5000/health/liveness
```

### Database Connection Failed

**Problem**: Service fails to connect to database

**Solutions**:
1. Check connection string format
2. Verify password (no special characters that need escaping)
3. Ensure database service is running and healthy
4. Check network connectivity

### Redis Connection Issues

**Problem**: Cache or message queue failures

**Solutions**:
1. Verify Redis is running: `docker compose ps redis`
2. Test connection: `docker compose exec redis redis-cli ping`
3. Check if password is required and correctly set

## Reference

- **Official Documentation**: https://docs.featbit.co/installation/docker-compose
- **Deployment Options**: https://docs.featbit.co/installation/deployment-options
