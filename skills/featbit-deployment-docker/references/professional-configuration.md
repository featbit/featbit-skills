# Professional Deployment - Complete Configuration

Complete guide for FeatBit Professional deployment with ClickHouse and Kafka for enterprise-scale deployments.

## Architecture Overview

Professional deployment provides:
- **Kafka**: High-throughput message queue for event streaming
- **ClickHouse**: High-performance columnar database for analytics
- **Massive scalability**: Handle millions of events per day
- **Advanced analytics**: Real-time data warehousing
- **Production-grade**: Enterprise reliability and performance

⚠️ **Complexity Warning**: Professional tier requires significant DevOps expertise and infrastructure resources.

## System Requirements

**Minimum**:
- RAM: 8GB
- CPU: 4 cores
- Disk: 50GB SSD

**Recommended**:
- RAM: 16GB+
- CPU: 8+ cores
- Disk: 100GB+ SSD
- Network: High bandwidth

## Official Configurations

FeatBit provides official docker-compose configurations for Professional deployment:

### Production Configuration
**File**: `docker-compose-pro.yml`  
**URL**: https://github.com/featbit/featbit/blob/main/docker-compose-pro.yml

This is the standard Professional deployment configuration with:
- Kafka for message queue
- ClickHouse for analytics
- PostgreSQL for primary storage
- Redis for caching

### High Availability Configuration
**File**: `docker-compose-pro-dev-HA-kafka.yml`  
**URL**: https://github.com/featbit/featbit/blob/main/docker/composes/docker-compose-pro-dev-HA-kafka.yml

This configuration provides:
- High Availability Kafka cluster
- Enhanced reliability
- Better fault tolerance
- Suitable for production environments requiring HA

## Service Configuration Reference

For detailed configuration of individual services (Redis, PostgreSQL, etc.), refer to:
- [standalone-configuration.md](./standalone-configuration.md) - Contains comprehensive service configuration examples
- [environment-variables.md](./environment-variables.md) - Complete environment variable reference

## Quick Start

### 1. Clone FeatBit Repository

```bash
# Clone the official FeatBit repository
git clone https://github.com/featbit/featbit.git
cd featbit
```

### 2. Choose Configuration

```bash
# Option 1: Standard Professional (recommended for most cases)
docker compose -f docker-compose-pro.yml up -d

# Option 2: High Availability with Kafka cluster
cd docker/composes
docker compose -f docker-compose-pro-dev-HA-kafka.yml up -d
```

### 3. Verify Deployment

```bash
# Check all services are running
docker compose ps

# View logs
docker compose logs -f

# Access the UI
# http://localhost:8081
```

## Configuration Customization

### Environment Variables

Create a `.env` file to customize deployment:

```bash
# Database
POSTGRES_PASSWORD=your_strong_password

# Redis (if authentication enabled)
REDIS_PASSWORD=your_redis_password

# ClickHouse
CLICKHOUSE_PASSWORD=your_clickhouse_password
```

Refer to [environment-variables.md](./environment-variables.md) for complete list.

### Service Scaling

```bash
# Scale evaluation server
docker compose -f docker-compose-pro.yml up -d --scale evaluation-server=3

# Scale API server
docker compose -f docker-compose-pro.yml up -d --scale api-server=2
```

## Production Considerations

### Using Managed Services

For production deployments, consider using managed services instead of containerized infrastructure:

**PostgreSQL**:
- AWS RDS
- Azure Database for PostgreSQL
- Google Cloud SQL

**Redis**:
- AWS ElastiCache
- Azure Cache for Redis
- Google Cloud Memorystore

**Kafka**:
- AWS MSK (Managed Streaming for Kafka)
- Confluent Cloud
- Azure Event Hubs

**ClickHouse**:
- ClickHouse Cloud
- Altinity.Cloud

Update connection strings in environment variables to point to managed services.

### Monitoring

Add monitoring tools to track system health:

```bash
# Kafka UI
docker run -d \
  -p 8090:8080 \
  -e KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka:9092 \
  provectuslabs/kafka-ui:latest

# ClickHouse monitoring
docker run -d \
  -p 8091:80 \
  spoonest/clickhouse-tabix-web-client:latest
```

## Troubleshooting

### Service Health Checks

```bash
# Check all services
docker compose ps

# Check specific service logs
docker compose logs kafka
docker compose logs clickhouse-server
docker compose logs postgresql

# Check resource usage
docker stats
```

### Common Issues

**Kafka not starting**: 
- Ensure sufficient memory allocated to Docker
- Check if ports 9092, 29092 are available
- View logs: `docker compose logs kafka`

**ClickHouse connection issues**:
- Verify ClickHouse is running: `docker compose ps clickhouse-server`
- Test connection: `docker compose exec clickhouse-server clickhouse-client --query "SELECT 1"`

**High memory usage**:
- Professional tier requires minimum 8GB RAM
- Allocate more resources to Docker Desktop/Engine
- Consider using managed services for infrastructure components

## Additional Resources

- **Official Documentation**: https://docs.featbit.co/installation/deployment-options
- **GitHub Repository**: https://github.com/featbit/featbit
- **Docker Compose Files**: https://github.com/featbit/featbit/tree/main/docker/composes
- **Kafka Documentation**: https://kafka.apache.org/documentation/
- **ClickHouse Documentation**: https://clickhouse.com/docs/
