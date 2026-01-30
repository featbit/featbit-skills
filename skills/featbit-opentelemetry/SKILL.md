---
name: featbit-opentelemetry
description: Expert knowledge of FeatBit's OpenTelemetry integration for observability including metrics, traces, and logs. Use when users ask about monitoring FeatBit services, setting up observability backends, or troubleshooting FeatBit performance.
appliesTo:
  - "**"
---

# FeatBit OpenTelemetry Observability Expert

You are an expert on FeatBit's OpenTelemetry integration. You have comprehensive knowledge of how FeatBit's backend services are instrumented for observability and can guide users on setting up metrics, traces, and logs.

## Overview

FeatBit's backend services (Api, Evaluation-Server, and Data Analytic Service) are fully instrumented with OpenTelemetry to publish observability metrics, traces, and logs. This enables comprehensive insights into FeatBit's performance and behavior.

## Core Knowledge Areas

### 1. Services & Instrumentation

**Instrumented Services:**
- **Api Service**: Written in .NET/C#, instrumented with [.NET Automatic Instrumentation](https://opentelemetry.io/docs/languages/net/automatic/)
- **Evaluation-Server**: Written in .NET/C#, instrumented with [.NET Automatic Instrumentation](https://opentelemetry.io/docs/languages/net/automatic/)
- **Data Analytic Service**: Written in Python, instrumented with [Python Automatic Instrumentation](https://opentelemetry.io/docs/languages/python/automatic/)

**Telemetry Data:**
- Metrics: CPU usage, memory consumption, network traffic, performance counters
- Traces: Request flows, service dependencies, latency breakdowns
- Logs: Application logs, error logs, debug information

### 2. Configuration

**Basic Required Environment Variables:**

All services need these environment variables to enable OpenTelemetry:

```bash
# Enable OpenTelemetry
ENABLE_OPENTELEMETRY=true

# Service identification (set appropriately for each service)
OTEL_SERVICE_NAME=featbit-api           # For Api service
OTEL_SERVICE_NAME=featbit-els           # For Evaluation-Server
OTEL_SERVICE_NAME=featbit-das           # For Data Analytic service

# Exporter endpoint (gRPC endpoint of OpenTelemetry collector)
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
```

**Additional Configuration:**

For **.NET/C# services** (Api and Evaluation-Server):
- Refer to [.NET OpenTelemetry configuration documentation](https://github.com/open-telemetry/opentelemetry-dotnet-instrumentation/blob/main/docs/config.md)
- Full list of supported environment variables available in the official docs

For **Python service** (Data Analytic):
- Refer to [Python OpenTelemetry configuration documentation](https://opentelemetry-python.readthedocs.io/en/latest/sdk/environment_variables.html)
- Complete environment variable reference in the official Python SDK docs

### 3. Ready-to-Run Example

FeatBit provides a complete working example with:
- **Seq**: For logs visualization
- **Jaeger**: For distributed tracing
- **Prometheus**: For metrics collection
- **OpenTelemetry Collector**: For telemetry data collection

**Quick Start Steps:**

```bash
# 1. Clone the FeatBit repository
git clone https://github.com/featbit/featbit.git
cd featbit

# 2. Build the test images
docker compose --project-directory . -f ./docker/composes/docker-compose-dev.yml build

# 3. Start OTEL collector, Seq, Jaeger, and Prometheus
docker compose --project-directory . -f ./docker/composes/docker-compose-otel-collector-contrib.yml up -d

# 4. Start FeatBit services with OpenTelemetry enabled
docker compose --project-directory . -f ./docker/composes/docker-compose-otel.yml up -d
```

**Access Observability Backends:**
- **Seq (Logs)**: http://localhost:8082/
- **Jaeger (Traces)**: http://localhost:16686/
- **Prometheus (Metrics)**: http://localhost:9090/

### 4. Common Use Cases

**Setting Up Observability:**
- Configure environment variables for each FeatBit service
- Deploy OpenTelemetry collector to receive telemetry data
- Connect to your observability backend (Seq, Jaeger, Prometheus, Datadog, New Relic, Grafana, etc.)

**Monitoring Performance:**
- Track request latency and throughput
- Monitor service health and availability
- Analyze resource utilization (CPU, memory, network)
- Set up alerts for anomalies

**Troubleshooting:**
- Use traces to identify bottlenecks in request flows
- Correlate logs with traces for debugging
- Analyze error rates and patterns
- Track down performance issues across services

**Production Deployment:**
- Use environment variables to configure telemetry endpoints
- Integrate with existing observability infrastructure
- Set appropriate service names for multi-environment deployments
- Configure sampling rates for high-volume scenarios

### 5. Integration with Other Observability Tools

FeatBit's OpenTelemetry integration works seamlessly with:
- **Datadog**: Enterprise monitoring and analytics platform
- **New Relic One**: Full-stack observability platform
- **Grafana**: Visualization and dashboards
- **Any OpenTelemetry-compatible backend**

## Best Practices

1. **Always set unique `OTEL_SERVICE_NAME`** for each service to distinguish telemetry data
2. **Use appropriate exporter endpoints** based on your infrastructure (http vs https, different ports)
3. **Start with the ready-to-run example** to understand the setup before customizing
4. **Configure sampling** in production to manage data volume
5. **Correlate metrics, traces, and logs** for comprehensive troubleshooting
6. **Monitor all three services** (Api, Evaluation-Server, Data Analytic) for complete visibility

## Documentation Reference

Official documentation: https://docs.featbit.co/integrations/observability/opentelemetry

## Related Topics

- DataDog Integration
- New Relic One Integration
- Grafana Integration
- FeatBit Architecture
- Deployment Options
