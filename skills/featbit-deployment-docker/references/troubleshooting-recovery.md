---
name: troubleshooting-recovery
description: Emergency recovery procedures, debugging tips, and help resources for FeatBit Docker deployments
---

# Troubleshooting: Recovery & Debugging

## Table of Contents

1. [Emergency Recovery](#emergency-recovery)
   - [Complete Reset](#complete-reset)
   - [Restore from Backup](#restore-from-backup)
2. [Debugging Tips](#debugging-tips)
   - [Enable Debug Logging](#enable-debug-logging)
   - [Access Container Shell](#access-container-shell)
   - [Network Debugging](#network-debugging)
   - [View Real-Time Logs](#view-real-time-logs)
3. [Getting Help](#getting-help)
4. [Reference](#reference)

## Emergency Recovery

### Complete Reset

If all else fails, completely reset the deployment:

```bash
# ⚠️ WARNING: This will delete ALL data!

# Stop all services
docker compose down

# Remove all volumes (data will be lost!)
docker compose down -v

# Remove all containers
docker rm -f $(docker ps -aq)

# Clean up Docker
docker system prune -a --volumes

# Start fresh
docker compose pull
docker compose up -d
```

### Restore from Backup

**PostgreSQL**:
```bash
# Stop services
docker compose down

# Remove old volume
docker volume rm featbit_postgres

# Start PostgreSQL
docker compose up -d postgresql

# Wait for PostgreSQL to be ready
sleep 10

# Restore backup
cat backup_20260201.sql | docker compose exec -T postgresql psql -U postgres featbit

# Start other services
docker compose up -d
```

**MongoDB**:
```bash
# Stop services
docker compose down

# Remove old volume
docker volume rm featbit_mongodb

# Start MongoDB
docker compose up -d mongodb

# Wait for MongoDB to be ready
sleep 10

# Restore backup
docker compose exec mongodb mongorestore /backup/featbit --db featbit

# Start other services
docker compose up -d
```

## Debugging Tips

### Enable Debug Logging

```yaml
api-server:
  environment:
    - Logging__LogLevel__Default=Debug
    - Logging__LogLevel__Microsoft=Information
```

### Access Container Shell

```bash
# Access container shell for debugging
docker compose exec api-server /bin/bash
docker compose exec postgresql /bin/bash

# Run commands inside container
docker compose exec api-server env  # View environment variables
docker compose exec api-server ps aux  # View processes
```

### Network Debugging

```bash
# Check network connectivity between containers
docker compose exec api-server ping postgresql
docker compose exec api-server nslookup postgresql
docker compose exec api-server curl http://redis:6379

# Inspect network
docker network inspect featbit_featbit-network
```

### View Real-Time Logs

```bash
# Follow logs for all services
docker compose logs -f

# Follow logs for specific service
docker compose logs -f api-server

# Last 100 lines
docker compose logs --tail=100 api-server

# Logs since specific time
docker compose logs --since 2024-02-01T10:00:00 api-server
```

## Getting Help

If you can't resolve the issue:

1. **Check official documentation**: https://docs.featbit.co
2. **Search GitHub issues**: https://github.com/featbit/featbit/issues
3. **Join Slack community**: https://join.slack.com/t/featbit/shared_invite/zt-1ew5e2vbb-x6Apan1xZOaYMnFzqZkGNQ
4. **Create GitHub issue** with:
   - FeatBit version
   - Docker Compose version
   - Deployment tier (Standalone/Standard/Professional)
   - Full docker-compose.yml (remove passwords!)
   - Error logs
   - Steps to reproduce

## Reference

- **Official Documentation**: https://docs.featbit.co/installation/docker-compose
- **GitHub Issues**: https://github.com/featbit/featbit/issues
- **FAQ**: https://docs.featbit.co/installation/faq
