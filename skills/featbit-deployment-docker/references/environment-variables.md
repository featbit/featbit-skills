---
name: environment-variables
description: Index and overview for all FeatBit Docker Compose environment variables. Links to detailed references for providers, database connections, service configuration, and OpenTelemetry.
---

# Environment Variables Reference

Complete reference for all FeatBit Docker Compose environment variables, organized into focused guides.

## Table of Contents

1. [Detailed Reference Guides](#detailed-reference-guides)
2. [Environment-Specific Configurations](#environment-specific-configurations)
3. [Quick Reference Table](#quick-reference-table)
4. [Troubleshooting Environment Variables](#troubleshooting-environment-variables)
5. [Reference](#reference)

## Detailed Reference Guides

### [Provider Configuration](env-vars-providers.md)

Variables that determine which infrastructure components FeatBit services use:
- **DbProvider** — Database type (`Postgres` | `MongoDb`)
- **MqProvider** — Message queue (`Postgres` | `Redis` | `Kafka`)
- **CacheProvider** — Caching layer (`None` | `Redis`)

### [Database Connection Strings](env-vars-database.md)

Connection string formats and examples for all supported backends:
- PostgreSQL (`Postgres__ConnectionString`)
- MongoDB (`MongoDb__ConnectionString`, `MongoDb__Database`)
- Redis (`Redis__ConnectionString`)
- Kafka (`Kafka__ConnectionString`)
- ClickHouse (`CLICKHOUSE_HOST`, `CLICKHOUSE_PORT`, etc.)

### [Service Configuration](env-vars-services.md)

UI, Data Analytics Server, and inter-service variables:
- UI configuration (`API_URL`, `EVALUATION_URL`, `DEMO_URL`, `BASE_HREF`)
- Data Analytics Server (`DB_PROVIDER` and per-database variables)
- Inter-service communication (`OLAP__ServiceHost`)
- Using `.env` files for secrets management

### [OpenTelemetry Configuration](env-vars-opentelemetry.md)

Observability setup for metrics, traces, and logs:
- Basic variables (`ENABLE_OPENTELEMETRY`, `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_ENDPOINT`)
- .NET and Python auto-instrumentation variables
- Complete docker-compose example with Seq, Jaeger, Prometheus
- Platform integration (Datadog, Grafana Cloud)
- Testing and troubleshooting OpenTelemetry

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
