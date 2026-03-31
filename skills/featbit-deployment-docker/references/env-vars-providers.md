---
name: env-vars-providers
description: FeatBit provider configuration variables (DbProvider, MqProvider, CacheProvider) that determine database, message queue, and cache infrastructure per deployment tier.
---

# Provider Configuration Variables

These variables determine which infrastructure components FeatBit services use.

## DbProvider

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

## MqProvider

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

## CacheProvider

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
