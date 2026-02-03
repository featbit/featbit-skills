---
name: featbit-opentelemetry
description: Expert guidance for setting up FeatBit's OpenTelemetry observability integration. Use when users ask about monitoring FeatBit, enabling metrics/traces/logs, configuring OTEL backends like Seq/Jaeger/Prometheus, or troubleshooting FeatBit performance.
appliesTo:
  - "**"
---

# FeatBit OpenTelemetry Integration

Guide users in setting up comprehensive observability for FeatBit's backend services using OpenTelemetry to publish metrics, traces, and logs.

## Overview

FeatBit's three backend services are fully instrumented with OpenTelemetry:

- **Api Service** (.NET/C#): [.NET Automatic Instrumentation](https://opentelemetry.io/docs/languages/net/automatic/)
- **Evaluation-Server** (.NET/C#): [.NET Automatic Instrumentation](https://opentelemetry.io/docs/languages/net/automatic/)
- **Data Analytic Service** (Python): [Python Automatic Instrumentation](https://opentelemetry.io/docs/languages/python/automatic/)

**What you get**: Metrics (CPU, memory, network), traces (request flows, latency), and logs (application events, errors).

## Quick Start Configuration

To enable OpenTelemetry, set these environment variables for each service:

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

**Additional configuration options**:
- .NET services: [Configuration docs](https://github.com/open-telemetry/opentelemetry-dotnet-instrumentation/blob/main/docs/config.md)
- Python service: [Environment variables](https://opentelemetry-python.readthedocs.io/en/latest/sdk/environment_variables.html)

## Ready-to-Run Example

Try the complete working example with Seq (logs), Jaeger (traces), and Prometheus (metrics):

```bash
# Clone the repository
git clone https://github.com/featbit/featbit.git
cd featbit

# Build the test images
docker compose --project-directory . -f ./docker/composes/docker-compose-dev.yml build

# Start OTEL collector, Seq, Jaeger, and Prometheus
docker compose --project-directory . -f ./docker/composes/docker-compose-otel-collector-contrib.yml up -d

# Start FeatBit services with OpenTelemetry enabled
docker compose --project-directory . -f ./docker/composes/docker-compose-otel.yml up -d
```

After starting, use FeatBit normally (create flags, evaluate, view insights), then access:
- **Seq (Logs)**: http://localhost:8082
- **Jaeger (Traces)**: http://localhost:16686
- **Prometheus (Metrics)**: http://localhost:9090

## Common Use Cases

**Setting Up Observability**:
1. Configure environment variables for each FeatBit service
2. Deploy OpenTelemetry collector to receive telemetry data
3. Connect to your backend (Seq, Jaeger, Prometheus, Datadog, New Relic, Grafana, or any OTEL-compatible system)

**Monitoring & Troubleshooting**:
- Track request latency, throughput, and resource utilization
- Use traces to identify bottlenecks in request flows
- Correlate logs with traces for debugging
- Set up alerts for performance anomalies

**Production Deployment**:
- Use unique `OTEL_SERVICE_NAME` for each service to distinguish telemetry
- Configure appropriate exporter endpoints (http/https, ports)
- Set sampling rates to manage data volume
- Monitor all three services for complete visibility

## Best Practices

1. **Always set unique `OTEL_SERVICE_NAME`** for each service
2. **Start with the ready-to-run example** to understand the setup before customizing
3. **Configure sampling in production** to manage telemetry data volume
4. **Correlate metrics, traces, and logs** for comprehensive troubleshooting

## References

- Official documentation: https://docs.featbit.co/integrations/observability/opentelemetry
- Compatible backends: Datadog, New Relic One, Grafana, any OpenTelemetry-compatible system
