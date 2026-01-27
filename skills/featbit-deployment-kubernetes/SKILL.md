---
name: featbit-deployment-kubernetes
description: Expert guidance for deploying FeatBit to Kubernetes using Helm Charts. Covers production deployments, service exposure, scaling, and cloud provider integrations (AKS, EKS, GKE). Use when users work with Kubernetes manifests or Helm values files.
appliesTo:
  - "**/values.yaml"
  - "**/values.yml"
  - "**/Chart.yaml"
  - "**/*-deployment.yaml"
  - "**/*-service.yaml"
  - "**/*-ingress.yaml"
  - "**/*-configmap.yaml"
  - "**/*-secret.yaml"
---

# FeatBit Kubernetes/Helm Deployment Expert

You are an expert on deploying FeatBit to Kubernetes clusters using Helm Charts, with deep knowledge of cloud-native deployments, service mesh, ingress controllers, and production-grade Kubernetes configurations.

## When to Use This Skill

Activate when users:
- Work with Kubernetes YAML manifests
- Use Helm values files (`values.yaml`)
- Ask about Kubernetes or Helm deployment
- Need guidance on AKS, EKS, GKE deployments
- Want to scale FeatBit in Kubernetes
- Ask about ingress, service mesh, or load balancers
- Need production-grade cloud-native setup

## Prerequisites

- **Kubernetes**: >= 1.23
- **Helm**: >= 3.7.0
- **kubectl**: Configured to connect to your cluster
- **Resources**: 4GB RAM minimum, 8GB recommended for production

## Quick Start: Helm Installation

### Step 1: Add FeatBit Helm Repository

```bash
helm repo add featbit https://featbit.github.io/featbit-charts/
helm repo update
```

### Step 2: Install FeatBit

```bash
# Create dedicated namespace
kubectl create namespace featbit

# Install with default values (development/testing)
helm install featbit featbit/featbit --namespace featbit

# Install with custom values (production)
helm install featbit featbit/featbit \
  -f your-values.yaml \
  --namespace featbit
```

### Step 3: Verify Installation

```bash
# Check pods status
kubectl get pods -n featbit

# Check services
kubectl get svc -n featbit

# Check ingress (if enabled)
kubectl get ingress -n featbit

# View logs
kubectl logs -n featbit -l app.kubernetes.io/name=featbit-api -f
```

## Architecture Tiers

FeatBit Helm chart supports three deployment tiers:

### Standalone (Development)
```yaml
architecture:
  tier: "standalone"
  database: "postgres"

postgresql:
  enabled: true

redis:
  enabled: false
```

### Standard (Production)
```yaml
architecture:
  tier: "standard"
  database: "postgres"  # or "mongodb"

postgresql:
  enabled: true

redis:
  enabled: true
```

### Professional (Enterprise)
```yaml
architecture:
  tier: "professional"
  database: "postgres"

postgresql:
  enabled: true

redis:
  enabled: true

kafka:
  enabled: true

clickhouse:
  enabled: true
```

## Service Exposure: 4 Methods

FeatBit requires exposing three services:
- **UI** (port 8081): Frontend web interface
- **API** (port 5000): Backend API server
- **Evaluation Server** (port 5100): Real-time feature flag evaluation with WebSocket support

### Method 1: Ingress (Recommended for Production)

**Prerequisites:**
- Ingress controller installed (NGINX, Traefik, etc.)
- DNS records pointing to ingress IP
- TLS certificates (Let's Encrypt via cert-manager recommended)

**Basic Ingress Configuration:**
```yaml
apiExternalUrl: "https://api.example.com"
evaluationServerExternalUrl: "https://eval.example.com"

global:
  ingressClassName: nginx

ui:
  ingress:
    enabled: true
    host: app.example.com

api:
  ingress:
    enabled: true
    host: api.example.com

els:
  ingress:
    enabled: true
    host: eval.example.com
```

**Ingress with TLS (Let's Encrypt):**
```yaml
apiExternalUrl: "https://api.example.com"
evaluationServerExternalUrl: "https://eval.example.com"

ui:
  ingress:
    enabled: true
    host: app.example.com
    tls:
      enabled: true
      secretName: featbit-tls-cert
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-prod"
      nginx.ingress.kubernetes.io/ssl-redirect: "true"

api:
  ingress:
    enabled: true
    host: api.example.com
    tls:
      enabled: true
      secretName: featbit-tls-cert
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-prod"

els:
  ingress:
    enabled: true
    host: eval.example.com
    tls:
      enabled: true
      secretName: featbit-tls-cert
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-prod"
      # WebSocket support
      nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
      nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
```

### Method 2: LoadBalancer (Cloud Environments)

**Static IP (Recommended):**
```yaml
apiExternalUrl: "http://API_EXTERNAL_IP:5000"
evaluationServerExternalUrl: "http://ELS_EXTERNAL_IP:5100"

ui:
  service:
    type: LoadBalancer
    loadBalancerIP: "UI_EXTERNAL_IP"  # Pre-allocated static IP

api:
  service:
    type: LoadBalancer
    loadBalancerIP: "API_EXTERNAL_IP"

els:
  service:
    type: LoadBalancer
    loadBalancerIP: "ELS_EXTERNAL_IP"
```

**Auto-Discovery (Dynamic IP):**
```yaml
apiExternalUrl: ""
evaluationServerExternalUrl: ""
autoDiscovery: true  # Chart will discover IPs automatically

ui:
  service:
    type: LoadBalancer

api:
  service:
    type: LoadBalancer

els:
  service:
    type: LoadBalancer
```

Get assigned IPs:
```bash
kubectl get svc -n featbit
```

### Method 3: NodePort (Development/Testing)

```yaml
apiExternalUrl: "http://NODE_IP:30050"
evaluationServerExternalUrl: "http://NODE_IP:30100"

ui:
  service:
    type: NodePort
    nodePort: 30025

api:
  service:
    type: NodePort
    nodePort: 30050

els:
  service:
    type: NodePort
    nodePort: 30100
```

Get node IP:
```bash
# For Minikube
minikube ip

# For cloud clusters
kubectl get nodes -o wide
```

### Method 4: Port Forward (Local Development)

```bash
# UI
kubectl port-forward service/featbit-ui 8081:8081 -n featbit

# API
kubectl port-forward service/featbit-api 5000:5000 -n featbit

# Evaluation Server
kubectl port-forward service/featbit-els 5100:5100 -n featbit
```

Access at: http://localhost:8081

## Using External Databases (Production Best Practice)

### External PostgreSQL

```yaml
postgresql:
  enabled: false

externalPostgresql:
  host: your-postgres-server.com
  port: 5432
  user: postgres
  database: featbit
  existingSecret: featbit-db-secret
  existingSecretPasswordKey: password
```

Create secret:
```bash
kubectl create secret generic featbit-db-secret \
  --from-literal=password='your_secure_password' \
  -n featbit
```

### External MongoDB

```yaml
mongodb:
  enabled: false

architecture:
  database: "mongodb"

externalMongodb:
  connectionString: "mongodb://username:password@mongo-server:27017"
  database: "featbit"
```

### External Redis

```yaml
redis:
  enabled: false

externalRedis:
  host: your-redis-server.com
  port: 6379
  password: "your_redis_password"
  # Or use existing secret
  existingSecret: featbit-redis-secret
  existingSecretPasswordKey: password
```

## Auto-Scaling Configuration

### Horizontal Pod Autoscaler (HPA)

**Requires metrics-server** installed in cluster.

```yaml
api:
  replicaCount: 2  # Minimum replicas
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 10
    targetCPUUtilizationPercentage: 80
    targetMemoryUtilizationPercentage: 80

els:
  replicaCount: 3  # More replicas for evaluation service
  autoscaling:
    enabled: true
    minReplicas: 3
    maxReplicas: 20
    targetCPUUtilizationPercentage: 75

ui:
  replicaCount: 2
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 5
    targetCPUUtilizationPercentage: 80
```

Check HPA status:
```bash
kubectl get hpa -n featbit
```

## Resource Management

### Resource Requests and Limits

```yaml
api:
  resources:
    requests:
      cpu: "500m"
      memory: "512Mi"
    limits:
      cpu: "2000m"
      memory: "2Gi"

els:
  resources:
    requests:
      cpu: "500m"
      memory: "512Mi"
    limits:
      cpu: "2000m"
      memory: "2Gi"

ui:
  resources:
    requests:
      cpu: "100m"
      memory: "128Mi"
    limits:
      cpu: "500m"
      memory: "512Mi"
```

## Production Deployment Example: Azure AKS

Complete guide with NGINX Ingress, cert-manager, and Azure Key Vault integration.

### Architecture
```
Internet → Azure Traffic Manager → Azure Load Balancer 
  → NGINX Ingress (TLS) → FeatBit Services
      ↓
  Azure Database for PostgreSQL + Azure Cache for Redis
```

### Prerequisites

```bash
# Install Azure CLI and kubectl
az aks install-cli

# Add Helm repositories
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add jetstack https://charts.jetstack.io
helm repo add featbit https://featbit.github.io/featbit-charts/
helm repo update

# Connect to AKS cluster
az aks get-credentials --resource-group <rg-name> --name <cluster-name>
```

### Step 1: Install NGINX Ingress Controller

```bash
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.replicaCount=3 \
  --set controller.service.externalTrafficPolicy=Local \
  --set controller.config.proxy-read-timeout="3600" \
  --set controller.config.proxy-send-timeout="3600"
```

Get Load Balancer IP:
```bash
kubectl get svc -n ingress-nginx ingress-nginx-controller
```

### Step 2: Install cert-manager

```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true \
  --version v1.16.2
```

### Step 3: Configure Let's Encrypt Issuer

Create `cluster-issuer.yaml`:
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

Apply:
```bash
kubectl apply -f cluster-issuer.yaml
```

### Step 4: Configure DNS

Point your domains to NGINX Ingress Load Balancer IP:
```
app.example.com  → A → <NGINX-LB-IP>
api.example.com  → A → <NGINX-LB-IP>
eval.example.com → A → <NGINX-LB-IP>
```

### Step 5: Create Azure Resources

```bash
# Create Azure Database for PostgreSQL
az postgres flexible-server create \
  --name featbit-pg \
  --resource-group <rg-name> \
  --location eastus \
  --admin-user featbitadmin \
  --admin-password <secure-password> \
  --sku-name Standard_B2s \
  --version 15

# Create Azure Cache for Redis
az redis create \
  --name featbit-redis \
  --resource-group <rg-name> \
  --location eastus \
  --sku Basic \
  --vm-size c0
```

### Step 6: Create Kubernetes Secrets

```bash
kubectl create secret generic featbit-postgres-secret \
  --from-literal=password='<postgres-password>' \
  -n featbit

kubectl create secret generic featbit-redis-secret \
  --from-literal=password='<redis-password>' \
  -n featbit
```

### Step 7: Deploy FeatBit with Custom Values

Create `featbit-aks-values.yaml`:
```yaml
apiExternalUrl: "https://api.example.com"
evaluationServerExternalUrl: "https://eval.example.com"

# Use external Azure Database for PostgreSQL
postgresql:
  enabled: false

externalPostgresql:
  host: featbit-pg.postgres.database.azure.com
  port: 5432
  user: featbitadmin
  database: featbit
  existingSecret: featbit-postgres-secret
  existingSecretPasswordKey: password

# Use Azure Cache for Redis
redis:
  enabled: false

externalRedis:
  host: featbit-redis.redis.cache.windows.net
  port: 6380
  tls: true
  existingSecret: featbit-redis-secret
  existingSecretPasswordKey: password

# Ingress with TLS
global:
  ingressClassName: nginx

ui:
  replicaCount: 2
  ingress:
    enabled: true
    host: app.example.com
    tls:
      enabled: true
      secretName: featbit-tls-cert
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-prod"
      nginx.ingress.kubernetes.io/ssl-redirect: "true"
  
  resources:
    requests:
      cpu: "100m"
      memory: "128Mi"
    limits:
      cpu: "500m"
      memory: "512Mi"

api:
  replicaCount: 2
  ingress:
    enabled: true
    host: api.example.com
    tls:
      enabled: true
      secretName: featbit-tls-cert
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-prod"
  
  resources:
    requests:
      cpu: "500m"
      memory: "512Mi"
    limits:
      cpu: "2000m"
      memory: "2Gi"
  
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 10
    targetCPUUtilizationPercentage: 80

els:
  replicaCount: 3
  ingress:
    enabled: true
    host: eval.example.com
    tls:
      enabled: true
      secretName: featbit-tls-cert
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-prod"
      nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
      nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
  
  resources:
    requests:
      cpu: "500m"
      memory: "512Mi"
    limits:
      cpu: "2000m"
      memory: "2Gi"
  
  autoscaling:
    enabled: true
    minReplicas: 3
    maxReplicas: 20
    targetCPUUtilizationPercentage: 75
```

Install:
```bash
helm install featbit featbit/featbit \
  -f featbit-aks-values.yaml \
  --namespace featbit
```

### Step 8: Verify Deployment

```bash
# Check pods
kubectl get pods -n featbit

# Check certificates
kubectl get certificate -n featbit

# Check ingress
kubectl get ingress -n featbit

# Test endpoints
curl https://api.example.com/health/liveness
```

## Upgrading FeatBit

### Check for Updates

```bash
helm repo update featbit

helm search repo featbit/featbit --versions
```

### Backup Before Upgrade

```bash
# Backup database (PostgreSQL example)
kubectl exec -n featbit postgresql-0 -- \
  pg_dump -U postgres featbit > backup-$(date +%Y%m%d).sql
```

### Perform Upgrade

```bash
helm upgrade featbit featbit/featbit \
  --version <target-version> \
  -f your-values.yaml \
  --namespace featbit
```

### Rollback if Needed

```bash
# View history
helm history featbit -n featbit

# Rollback to previous version
helm rollback featbit -n featbit
```

## Monitoring and Observability

### Prometheus Integration

```yaml
# Enable ServiceMonitor (requires Prometheus Operator)
api:
  metrics:
    enabled: true
    serviceMonitor:
      enabled: true

els:
  metrics:
    enabled: true
    serviceMonitor:
      enabled: true
```

### View Logs

```bash
# All pods
kubectl logs -n featbit -l app.kubernetes.io/instance=featbit -f

# Specific service
kubectl logs -n featbit -l app.kubernetes.io/name=featbit-api -f

# Tail logs from all replicas
kubectl logs -n featbit -l app.kubernetes.io/name=featbit-els -f --all-containers=true
```

### Health Checks

```bash
# Check liveness
kubectl get pods -n featbit -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.conditions[?(@.type=="Ready")].status}{"\n"}{end}'

# Port forward and test
kubectl port-forward -n featbit svc/featbit-api 5000:5000
curl http://localhost:5000/health/liveness
```

## Troubleshooting

### Pods Not Starting

```bash
# Check pod status
kubectl describe pod -n featbit <pod-name>

# Check events
kubectl get events -n featbit --sort-by='.lastTimestamp'

# Check logs
kubectl logs -n featbit <pod-name> --previous
```

### Ingress Not Working

```bash
# Check ingress
kubectl describe ingress -n featbit

# Check ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller -f

# Test from inside cluster
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -- \
  curl http://featbit-api.featbit.svc.cluster.local:5000/health
```

### Certificate Issues

```bash
# Check certificate status
kubectl describe certificate -n featbit featbit-tls-cert

# Check cert-manager logs
kubectl logs -n cert-manager -l app=cert-manager -f

# Check certificate request
kubectl describe certificaterequest -n featbit
```

### Database Connection Issues

```bash
# Test connection from pod
kubectl run -it --rm debug --image=postgres:15 --restart=Never -- \
  psql -h <db-host> -U <username> -d featbit

# Check secrets
kubectl get secret -n featbit featbit-db-secret -o jsonpath='{.data.password}' | base64 -d
```

## Multi-Region Deployment (Azure Traffic Manager)

For high availability across multiple regions:

1. Deploy FeatBit to multiple AKS clusters (different regions)
2. Configure Azure Traffic Manager with performance routing
3. Use shared Azure Database for PostgreSQL with read replicas
4. Configure Redis with geo-replication

See detailed guide: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples/aks

## Best Practices

### Security
- ✅ Use external managed databases (Azure Database, AWS RDS)
- ✅ Store secrets in Kubernetes Secrets or Azure Key Vault
- ✅ Enable TLS for all ingress endpoints
- ✅ Use network policies to restrict pod-to-pod traffic
- ✅ Regularly update container images

### Reliability
- ✅ Use multiple replicas for all services
- ✅ Configure resource requests and limits
- ✅ Enable auto-scaling (HPA)
- ✅ Set up proper health checks
- ✅ Use pod disruption budgets

### Performance
- ✅ Use external Redis for caching (Standard/Professional tiers)
- ✅ Scale evaluation server based on traffic
- ✅ Enable connection pooling for databases
- ✅ Use CDN for UI static assets
- ✅ Monitor and adjust resource limits

### Operations
- ✅ Backup databases regularly
- ✅ Test upgrades in non-production environment first
- ✅ Use Infrastructure as Code (Helm values in Git)
- ✅ Set up monitoring and alerting
- ✅ Document your configuration

## Helm Chart Resources

- **Chart Repository**: https://github.com/featbit/featbit-charts
- **Chart README**: https://github.com/featbit/featbit-charts/blob/main/README.md
- **Examples**: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples
- **AKS Deployment Guide**: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples/aks
- **Current Version**: 0.9.0 (FeatBit App: 5.2.0)
- **Compatibility**: Kubernetes >= 1.23, Helm >= 3.7.0

For detailed configuration options, advanced deployment patterns, and troubleshooting, refer to the official chart repository.

## Official Resources

- **Helm Charts**: https://github.com/featbit/featbit-charts
- **Main Repository**: https://github.com/featbit/featbit
- **Documentation**: https://docs.featbit.co
