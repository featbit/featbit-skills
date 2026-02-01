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

## OpenTelemetry Configuration

FeatBit's backend services (Api, Evaluation-Server, and Data Analytic service) are instrumented with OpenTelemetry to publish observability metrics, traces, and logs.

### Basic OpenTelemetry Variables

**Required variables** to enable OpenTelemetry for any service:

| Variable | Description | Example |
|----------|-------------|---------|
| `ENABLE_OPENTELEMETRY` | Enable/disable OpenTelemetry | `true` or `false` |
| `OTEL_SERVICE_NAME` | Service identifier in observability backends | `featbit-api`, `featbit-els`, `featbit-das` |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | gRPC endpoint of OpenTelemetry exporter | `http://otel-collector:4317` |

**Examples**:

```yaml
# API Server with OpenTelemetry
api-server:
  environment:
    - ENABLE_OPENTELEMETRY=true
    - OTEL_SERVICE_NAME=featbit-api
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317

# Evaluation Server with OpenTelemetry
evaluation-server:
  environment:
    - ENABLE_OPENTELEMETRY=true
    - OTEL_SERVICE_NAME=featbit-els
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317

# Data Analytics Server with OpenTelemetry
da-server:
  environment:
    - ENABLE_OPENTELEMETRY=true
    - OTEL_SERVICE_NAME=featbit-das
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
```

### Recommended Service Names

Use these consistent service names for easier identification:

- **API Server**: `featbit-api`
- **Evaluation Server**: `featbit-els`
- **Data Analytics Server**: `featbit-das`

### Additional OpenTelemetry Variables

**API Server and Evaluation Server (.NET/C# services)**:

These services support [.NET Automatic Instrumentation environment variables](https://github.com/open-telemetry/opentelemetry-dotnet-instrumentation/blob/main/docs/config.md).

Common additional variables:
```yaml
api-server:
  environment:
    - ENABLE_OPENTELEMETRY=true
    - OTEL_SERVICE_NAME=featbit-api
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
    # Additional configuration
    - OTEL_EXPORTER_OTLP_PROTOCOL=grpc  # or http/protobuf
    - OTEL_TRACES_SAMPLER=always_on     # Sampling strategy
    - OTEL_METRICS_EXPORTER=otlp        # Metrics exporter
    - OTEL_LOGS_EXPORTER=otlp           # Logs exporter
```

**Data Analytics Server (Python service)**:

This service supports [Python Automatic Instrumentation environment variables](https://opentelemetry-python.readthedocs.io/en/latest/sdk/environment_variables.html).

Common additional variables:
```yaml
da-server:
  environment:
    - ENABLE_OPENTELEMETRY=true
    - OTEL_SERVICE_NAME=featbit-das
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
    # Additional configuration
    - OTEL_EXPORTER_OTLP_PROTOCOL=grpc
    - OTEL_PYTHON_LOGGING_AUTO_INSTRUMENTATION_ENABLED=true
```

### Complete OpenTelemetry Example

Example configuration with Seq (logs), Jaeger (traces), and Prometheus (metrics):

```yaml
version: '3'

services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    command: ["--config=/etc/otel-collector-config.yaml"]
    volumes:
      - ./otel-collector-config.yaml:/etc/otel-collector-config.yaml
    ports:
      - "4317:4317"   # OTLP gRPC receiver
      - "4318:4318"   # OTLP HTTP receiver

  api-server:
    image: featbit/featbit-api-server:latest
    environment:
      # ... other variables ...
      - ENABLE_OPENTELEMETRY=true
      - OTEL_SERVICE_NAME=featbit-api
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
    depends_on:
      - otel-collector

  evaluation-server:
    image: featbit/featbit-evaluation-server:latest
    environment:
      # ... other variables ...
      - ENABLE_OPENTELEMETRY=true
      - OTEL_SERVICE_NAME=featbit-els
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
    depends_on:
      - otel-collector

  da-server:
    image: featbit/featbit-data-analytics-server:latest
    environment:
      # ... other variables ...
      - ENABLE_OPENTELEMETRY=true
      - OTEL_SERVICE_NAME=featbit-das
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
    depends_on:
      - otel-collector

  # Observability backends
  seq:
    image: datalust/seq:latest
    environment:
      - ACCEPT_EULA=Y
    ports:
      - "8082:80"     # Seq UI

  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686" # Jaeger UI

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"   # Prometheus UI
```

### Integration with Popular Observability Platforms

**Important**: FeatBit services only need the basic OpenTelemetry variables (`ENABLE_OPENTELEMETRY`, `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_ENDPOINT`). Integration with specific observability platforms is configured in the **OpenTelemetry Collector**, not in FeatBit services.

**Architecture**:
```
FeatBit Services → OpenTelemetry Collector → Observability Platform
                   (configure endpoint)    (configure platform credentials)
```

**FeatBit Service Configuration** (same for all platforms):
```yaml
api-server:
  environment:
    - ENABLE_OPENTELEMETRY=true
    - OTEL_SERVICE_NAME=featbit-api
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
```

**Platform Integration** is done in OpenTelemetry Collector configuration file:

**Example: Datadog Integration**

1. FeatBit services send to collector (configuration above)
2. Configure collector to forward to Datadog:

```yaml
# otel-collector-config.yaml
exporters:
  datadog:
    api:
      key: ${DD_API_KEY}
      site: datadoghq.com

service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [datadog]
    metrics:
      receivers: [otlp]
      exporters: [datadog]
```

**Example: Grafana Cloud Integration**

```yaml
# otel-collector-config.yaml
exporters:
  otlphttp:
    endpoint: https://otlp-gateway-prod-us-central-0.grafana.net/otlp
    headers:
      authorization: Basic ${GRAFANA_CLOUD_API_KEY}

service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [otlphttp]
```

**For detailed platform integration**, refer to:
- **Datadog**: https://docs.featbit.co/integrations/observability/datadog
- **New Relic**: https://docs.featbit.co/integrations/observability/newrelic
- **Grafana**: https://docs.featbit.co/integrations/observability/grafana
- **OpenTelemetry Collector Configuration**: https://opentelemetry.io/docs/collector/configuration/

### Testing OpenTelemetry Setup

After enabling OpenTelemetry, verify it's working:

1. **Check service logs**:
```bash
docker compose logs api-server | grep -i "opentelemetry"
docker compose logs api-server | grep -i "otel"
```

2. **Verify collector is receiving data**:
```bash
docker compose logs otel-collector
```

3. **Access observability backends**:
- Seq (logs): http://localhost:8082
- Jaeger (traces): http://localhost:16686
- Prometheus (metrics): http://localhost:9090

### Troubleshooting OpenTelemetry

**Issue**: No telemetry data appearing

**Solutions**:
1. Verify `ENABLE_OPENTELEMETRY=true` is set
2. Check collector endpoint is accessible from service
3. Check service logs for OpenTelemetry errors
4. Verify collector configuration is correct

**Issue**: Partial data (only traces, no metrics/logs)

**Solutions**:
1. Check exporter configuration for each signal type
2. Verify collector is configured to receive all signal types
3. Check backend supports all signal types

### References

- **FeatBit OpenTelemetry Documentation**: https://docs.featbit.co/integrations/observability/opentelemetry
- **.NET Auto Instrumentation Config**: https://github.com/open-telemetry/opentelemetry-dotnet-instrumentation/blob/main/docs/config.md
- **Python Auto Instrumentation Config**: https://opentelemetry-python.readthedocs.io/en/latest/sdk/environment_variables.html
- **OpenTelemetry Collector**: https://opentelemetry.io/docs/collector/

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
