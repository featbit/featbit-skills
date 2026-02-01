# Production Best Practices

Comprehensive guide for deploying FeatBit with Docker Compose in production environments.

## Security

### 1. Change All Default Passwords

**Critical**: Never use default passwords in production.

```yaml
postgresql:
  environment:
    POSTGRES_PASSWORD: ${DB_PASSWORD}  # Use strong password

mongodb:
  environment:
    MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD}  # Use strong password

clickhouse-server:
  environment:
    CLICKHOUSE_PASSWORD: ${CLICKHOUSE_PASSWORD}  # Use strong password
```

Generate strong passwords:
```bash
# Generate random password
openssl rand -base64 32

# Or use pwgen
pwgen -s 32 1
```

### 2. Use Environment Files for Secrets

```bash
# Create .env file (add to .gitignore!)
cat > .env <<'EOF'
DB_PASSWORD=<generated_strong_password>
MONGO_PASSWORD=<generated_strong_password>
REDIS_PASSWORD=<generated_strong_password>
CLICKHOUSE_PASSWORD=<generated_strong_password>
EOF

# Protect the file
chmod 600 .env
```

### 3. Use Docker Secrets (Swarm Mode)

For Docker Swarm deployments:

```yaml
services:
  api-server:
    secrets:
      - db_password
    environment:
      - Postgres__ConnectionString=Host=postgresql;Port=5432;Username=postgres;Password_File=/run/secrets/db_password;Database=featbit

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### 4. Network Isolation

Separate frontend and backend networks:

```yaml
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access

services:
  ui:
    networks:
      - frontend  # Accessible from outside
  
  api-server:
    networks:
      - frontend  # Accessible from outside
      - backend   # Access to databases
  
  evaluation-server:
    networks:
      - frontend  # WebSocket access from browsers
      - backend   # Access to databases
  
  postgresql:
    networks:
      - backend   # Only accessible from backend network
  
  redis:
    networks:
      - backend
  
  mongodb:
    networks:
      - backend
```

### 5. Enable SSL/TLS

#### Option A: Reverse Proxy with nginx

```yaml
nginx:
  image: nginx:alpine
  ports:
    - "443:443"
    - "80:80"
  volumes:
    - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    - ./nginx/ssl:/etc/nginx/ssl:ro
  depends_on:
    - ui
    - api-server
    - evaluation-server
  networks:
    - frontend
```

**nginx.conf**:
```nginx
# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name featbit.yourdomain.com;
    return 301 https://$host$request_uri;
}

# UI
server {
    listen 443 ssl http2;
    server_name featbit.yourdomain.com;
    
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    location / {
        proxy_pass http://ui:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# API Server
server {
    listen 443 ssl http2;
    server_name api.featbit.yourdomain.com;
    
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    location / {
        proxy_pass http://api-server:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Evaluation Server
server {
    listen 443 ssl http2;
    server_name eval.featbit.yourdomain.com;
    
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    location / {
        proxy_pass http://evaluation-server:5100;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 86400;
    }
}
```

#### Option B: Let's Encrypt with Traefik

```yaml
traefik:
  image: traefik:v2.10
  command:
    - "--providers.docker=true"
    - "--entrypoints.web.address=:80"
    - "--entrypoints.websecure.address=:443"
    - "--certificatesresolvers.letsencrypt.acme.httpchallenge=true"
    - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
    - "--certificatesresolvers.letsencrypt.acme.email=admin@yourdomain.com"
    - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
  ports:
    - "80:80"
    - "443:443"
  volumes:
    - "/var/run/docker.sock:/var/run/docker.sock:ro"
    - "./letsencrypt:/letsencrypt"
  networks:
    - frontend

ui:
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.ui.rule=Host(`featbit.yourdomain.com`)"
    - "traefik.http.routers.ui.entrypoints=websecure"
    - "traefik.http.routers.ui.tls.certresolver=letsencrypt"
```

### 6. Firewall Configuration

```bash
# Ubuntu/Debian with UFW
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp    # SSH
ufw allow 80/tcp    # HTTP
ufw allow 443/tcp   # HTTPS
ufw deny 5432/tcp   # Block direct PostgreSQL access
ufw deny 27017/tcp  # Block direct MongoDB access
ufw deny 6379/tcp   # Block direct Redis access
ufw enable

# Check status
ufw status
```

## High Availability

### 1. Health Checks

Enable health checks for all critical services:

```yaml
api-server:
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:5000/health/liveness"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 40s
  restart: unless-stopped

evaluation-server:
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:5100/health/liveness"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 40s
  restart: unless-stopped

postgresql:
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U postgres"]
    interval: 5s
    timeout: 5s
    retries: 5
  restart: unless-stopped

redis:
  healthcheck:
    test: ["CMD", "redis-cli", "ping"]
    interval: 5s
    timeout: 3s
    retries: 5
  restart: unless-stopped
```

### 2. Resource Limits

Prevent any single container from consuming all resources:

```yaml
api-server:
  deploy:
    resources:
      limits:
        cpus: '2'
        memory: 2G
      reservations:
        cpus: '1'
        memory: 512M

evaluation-server:
  deploy:
    resources:
      limits:
        cpus: '2'
        memory: 2G
      reservations:
        cpus: '1'
        memory: 1G

postgresql:
  deploy:
    resources:
      limits:
        cpus: '4'
        memory: 4G
      reservations:
        cpus: '2'
        memory: 2G

redis:
  deploy:
    resources:
      limits:
        cpus: '1'
        memory: 1G
      reservations:
        cpus: '0.5'
        memory: 512M
```

### 3. Use Managed Services

For production, strongly recommend using managed services:

**Databases**:
- AWS RDS (PostgreSQL)
- Azure Database for PostgreSQL
- MongoDB Atlas
- AWS DocumentDB

**Caching**:
- AWS ElastiCache (Redis)
- Azure Cache for Redis
- Redis Enterprise Cloud

**Message Queue**:
- AWS MSK (Kafka)
- Confluent Cloud
- Azure Event Hubs

**Analytics**:
- ClickHouse Cloud
- AWS RDS for ClickHouse

**Benefits**:
- Automated backups and point-in-time recovery
- Automatic failover and high availability
- Managed updates and patching
- Built-in monitoring and alerting
- Better security and compliance
- Professional support

## Backup and Recovery

### Automated PostgreSQL Backups

```bash
#!/bin/bash
# backup-postgres.sh

BACKUP_DIR=/backups/postgres
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE=$BACKUP_DIR/featbit_$DATE.sql

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup
docker compose exec -T postgresql pg_dump -U postgres featbit > $BACKUP_FILE

# Compress
gzip $BACKUP_FILE

# Keep only last 30 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

echo "Backup completed: ${BACKUP_FILE}.gz"
```

Schedule with cron:
```bash
# Run daily at 2 AM
0 2 * * * /path/to/backup-postgres.sh
```

### Automated MongoDB Backups

```bash
#!/bin/bash
# backup-mongodb.sh

BACKUP_DIR=/backups/mongodb
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_PATH=$BACKUP_DIR/featbit_$DATE

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup
docker compose exec -T mongodb mongodump \
  --username admin \
  --password $(cat .env | grep MONGO_PASSWORD | cut -d '=' -f2) \
  --db featbit \
  --out /backup/$DATE

docker compose cp mongodb:/backup/$DATE $BACKUP_PATH

# Compress
tar czf ${BACKUP_PATH}.tar.gz -C $BACKUP_DIR $(basename $BACKUP_PATH)
rm -rf $BACKUP_PATH

# Keep only last 30 days
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: ${BACKUP_PATH}.tar.gz"
```

### Volume Snapshots

```bash
#!/bin/bash
# snapshot-volumes.sh

SNAPSHOT_DIR=/backups/volumes
DATE=$(date +%Y%m%d_%H%M%S)

# Stop services
docker compose down

# Backup PostgreSQL volume
docker run --rm \
  -v featbit_postgres:/data \
  -v $SNAPSHOT_DIR:/backup \
  alpine tar czf /backup/postgres_$DATE.tar.gz /data

# Backup MongoDB volume
docker run --rm \
  -v featbit_mongodb:/data \
  -v $SNAPSHOT_DIR:/backup \
  alpine tar czf /backup/mongodb_$DATE.tar.gz /data

# Backup Redis volume
docker run --rm \
  -v featbit_redis:/data \
  -v $SNAPSHOT_DIR:/backup \
  alpine tar czf /backup/redis_$DATE.tar.gz /data

# Start services
docker compose up -d

echo "Volume snapshots completed"
```

## Monitoring

### Add Prometheus and Grafana

```yaml
prometheus:
  image: prom/prometheus:latest
  volumes:
    - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
    - prometheus_data:/prometheus
  ports:
    - "9090:9090"
  networks:
    - monitoring
  restart: unless-stopped

grafana:
  image: grafana/grafana:latest
  environment:
    - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
    - GF_INSTALL_PLUGINS=grafana-clock-panel
  volumes:
    - grafana_data:/var/lib/grafana
    - ./grafana/dashboards:/etc/grafana/provisioning/dashboards:ro
    - ./grafana/datasources:/etc/grafana/provisioning/datasources:ro
  ports:
    - "3000:3000"
  networks:
    - monitoring
  restart: unless-stopped
```

**prometheus.yml**:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['cadvisor:8080']
  
  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']
```

### Container Metrics with cAdvisor

```yaml
cadvisor:
  image: gcr.io/cadvisor/cadvisor:latest
  container_name: cadvisor
  volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:ro
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
  ports:
    - "8080:8080"
  networks:
    - monitoring
  restart: unless-stopped
```

### Logging with ELK Stack

For centralized logging:

```yaml
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
  environment:
    - discovery.type=single-node
    - xpack.security.enabled=false
  volumes:
    - elasticsearch_data:/usr/share/elasticsearch/data
  networks:
    - logging

logstash:
  image: docker.elastic.co/logstash/logstash:8.11.0
  volumes:
    - ./logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
  depends_on:
    - elasticsearch
  networks:
    - logging

kibana:
  image: docker.elastic.co/kibana/kibana:8.11.0
  ports:
    - "5601:5601"
  depends_on:
    - elasticsearch
  networks:
    - logging
```

## Performance Optimization

### PostgreSQL Tuning

```yaml
postgresql:
  command: >
    postgres
    -c max_connections=200
    -c shared_buffers=512MB
    -c effective_cache_size=2GB
    -c maintenance_work_mem=128MB
    -c checkpoint_completion_target=0.9
    -c wal_buffers=16MB
    -c default_statistics_target=100
    -c random_page_cost=1.1
    -c effective_io_concurrency=200
    -c work_mem=4MB
    -c min_wal_size=1GB
    -c max_wal_size=4GB
```

### Redis Tuning

```yaml
redis:
  command: >
    redis-server
    --maxmemory 1gb
    --maxmemory-policy allkeys-lru
    --save 60 1000
    --appendonly yes
    --appendfsync everysec
    --tcp-backlog 511
    --timeout 300
```

### MongoDB Tuning

```yaml
mongodb:
  command: >
    mongod
    --wiredTigerCacheSizeGB 2
    --maxConns 200
    --logRotate reopen
```

## Upgrade Strategy

### Zero-Downtime Rolling Update

```bash
#!/bin/bash
# rolling-update.sh

# Pull latest images
docker compose pull

# Update services one by one
for service in evaluation-server api-server ui da-server; do
  echo "Updating $service..."
  docker compose up -d --no-deps --force-recreate $service
  
  # Wait for health check
  sleep 30
  
  # Verify service is healthy
  docker compose ps $service
done

echo "Update completed"
```

### Blue-Green Deployment

```bash
# Deploy new version (green)
docker compose -f docker-compose-green.yml up -d

# Test green deployment
curl http://localhost:9081/health

# Switch traffic (update nginx/traefik config)
# ...

# Remove old version (blue)
docker compose -f docker-compose-blue.yml down
```

## Disaster Recovery Plan

### 1. Regular Testing

Test backups monthly:
```bash
# Restore to test environment
docker compose -f docker-compose-test.yml down -v
cat backup_20260201.sql | docker compose -f docker-compose-test.yml exec -T postgresql psql -U postgres featbit
docker compose -f docker-compose-test.yml up -d
```

### 2. Document Recovery Procedures

Create runbook with:
- Backup locations
- Restore commands
- Contact information
- Escalation procedures

### 3. Multi-Region Backups

Store backups in multiple locations:
```bash
# Upload to S3
aws s3 cp backup_20260201.sql.gz s3://featbit-backups/$(date +%Y/%m/%d)/

# Upload to Azure Blob Storage
az storage blob upload --file backup_20260201.sql.gz --container backups --name $(date +%Y/%m/%d)/backup.sql.gz
```

## Compliance and Security

### 1. Log Access

```yaml
api-server:
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "3"
```

### 2. Regular Security Updates

```bash
# Update base images weekly
docker compose pull
docker compose up -d
```

### 3. Vulnerability Scanning

```bash
# Scan images for vulnerabilities
docker scan featbit/featbit-api-server:latest
```

## Reference

- **Official Documentation**: https://docs.featbit.co/installation/docker-compose
- **Docker Compose Best Practices**: https://docs.docker.com/compose/production/
- **Security Best Practices**: https://docs.docker.com/engine/security/
