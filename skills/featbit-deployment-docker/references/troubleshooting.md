---
name: troubleshooting
description: Index for FeatBit Docker Compose troubleshooting guides. Routes to focused guides by category - login, database, networking, infrastructure, and recovery.
---

# Troubleshooting Guide

Troubleshooting index for FeatBit Docker Compose deployments. Each category links to a focused guide.

## Login & Authentication

Problems with login, database initialization, and API server health.

**[troubleshooting-login.md](troubleshooting-login.md)**

| Issue | Symptom |
|-------|---------|
| Login Failed / Cannot Sign In | Unable to log in with default credentials, login page errors |
| Database Not Initialized | Empty database, no seed data for authentication |
| API Server Not Running | Login requests fail due to crashed or missing API server |

## Database Issues

Problems with database connections, authentication, and data persistence.

**[troubleshooting-database.md](troubleshooting-database.md)**

| Issue | Symptom |
|-------|---------|
| Database Connection Failed | "connection refused" or "authentication failed" on startup |
| MongoDB Authentication Failed | "Authentication failed" connecting to MongoDB |
| Data Loss After Restart | Data disappears after stopping and restarting containers |

## Networking Issues

Problems with ports, UI-API connectivity, WebSocket connections, Redis, and container health.

**[troubleshooting-networking.md](troubleshooting-networking.md)**

| Issue | Symptom |
|-------|---------|
| Port Already in Use | "port is already allocated" error on startup |
| UI Cannot Connect to API | UI loads but shows "Cannot connect to API" or "Network Error" |
| WebSocket Connection Failed | UI can't establish WebSocket connection to evaluation server |
| Redis Connection Issues | "Redis connection failed" or "ECONNREFUSED redis:6379" |
| Container Healthy but Not Responding | Health check passes but service doesn't respond |

## Infrastructure Issues

Problems with service startup, Kafka, ClickHouse, disk space, and performance.

**[troubleshooting-infrastructure.md](troubleshooting-infrastructure.md)**

| Issue | Symptom |
|-------|---------|
| Service Won't Start | Service shows "Exit 1" or constantly restarting |
| Kafka Won't Start | Kafka service constantly restarting (Professional tier) |
| ClickHouse Connection Failed | Data Analytics server can't connect to ClickHouse |
| Disk Space Issues | "No space left on device" errors |
| Performance Issues | Slow response times, high latency |

## Recovery & Debugging

Emergency recovery procedures, debugging techniques, and support resources.

**[troubleshooting-recovery.md](troubleshooting-recovery.md)**

| Topic | Description |
|-------|-------------|
| Complete Reset | Full reset when all else fails (destroys data) |
| Restore from Backup | PostgreSQL and MongoDB backup restoration |
| Debug Logging | Enable verbose logging for diagnosis |
| Container Shell Access | Interactive debugging inside containers |
| Network Debugging | Container-to-container connectivity checks |
| Getting Help | Official docs, GitHub issues, Slack community |
