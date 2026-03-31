---
name: env-vars-database
description: FeatBit database and cache connection string formats and examples for PostgreSQL, MongoDB, Redis, Kafka, and ClickHouse.
---

# Database Connection Strings

## Table of Contents

1. [PostgreSQL](#postgresql)
2. [MongoDB](#mongodb)
3. [Redis](#redis)
4. [Kafka](#kafka)
5. [ClickHouse](#clickhouse-data-analytics-server-only)

## PostgreSQL

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

## MongoDB

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

## Redis

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

## Kafka

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

## ClickHouse (Data Analytics Server only)

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
