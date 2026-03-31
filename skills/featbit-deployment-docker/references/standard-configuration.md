---
name: standard-configuration
description: Overview and index for FeatBit Standard deployment options - PostgreSQL + Redis or MongoDB + Redis. Links to detailed configuration files.
---

# Standard Deployment - Overview

Standard deployment adds Redis for caching and message queuing on top of a database backend, providing:
- **Better performance**: Redis caching reduces database load
- **Improved reliability**: Dedicated message queue
- **Production-ready**: Suitable for medium to large teams
- **Scalability**: Can handle high-volume traffic

## Choose Your Database Backend

| Aspect | PostgreSQL + Redis | MongoDB + Redis |
|--------|-------------------|-----------------|
| **Config file** | `docker-compose-standard.yml` | `docker-compose-mongodb.yml` |
| **DbProvider** | `Postgres` | `MongoDb` |
| **Schema** | Relational, SQL-based | Document-oriented, flexible schema |
| **Init scripts** | `.sql` files, auto-executed | `.js` files, auto-executed |
| **Best for** | Teams familiar with PostgreSQL | Document-oriented workloads |

### Option A: PostgreSQL + Redis

Full docker-compose configuration, initialization scripts, managed service examples (AWS RDS, Azure Database for PostgreSQL).

**Detailed guide**: [standard-postgresql-configuration.md](standard-postgresql-configuration.md)

**Quick start**:
```bash
git clone https://github.com/featbit/featbit.git
cd featbit
docker compose -f docker-compose-standard.yml up -d
```

### Option B: MongoDB + Redis

Full docker-compose configuration, initialization scripts, MongoDB-specific setup.

**Detailed guide**: [standard-mongodb-configuration.md](standard-mongodb-configuration.md)

**Quick start**:
```bash
git clone https://github.com/featbit/featbit.git
cd featbit
docker compose -f docker-compose-mongodb.yml up -d
```

## Resource Requirements

**Minimum**: 4GB RAM, 2 CPU cores, 20GB disk

**Recommended for Production**: 8GB RAM, 4 CPU cores, 50GB+ SSD

## Reference

- **Official Documentation**: https://docs.featbit.co/installation/docker-compose
- **Deployment Options Comparison**: https://docs.featbit.co/installation/deployment-options
