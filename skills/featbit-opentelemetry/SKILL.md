---
name: featbit-opentelemetry
description: Expert guidance for setting up FeatBit's OpenTelemetry observability integration. Use when users ask about monitoring FeatBit, enabling metrics/traces/logs, configuring OTEL backends like Seq/Jaeger/Prometheus, or troubleshooting FeatBit performance. Do not use for application-level SDK instrumentation or Azure Monitor/Application Insights questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: platform
---

# FeatBit OpenTelemetry Integration

Set up comprehensive observability for FeatBit's backend services using OpenTelemetry to publish metrics, traces, and logs.

## Execution Procedure

```
1. DETERMINE deployment type
   - Local dev → use ready-to-run Docker Compose example
   - Production → configure env vars per service manually

2. CONFIGURE environment variables for each service
   - Set ENABLE_OPENTELEMETRY=true
   - Assign unique OTEL_SERVICE_NAME per service
   - Set OTEL_EXPORTER_OTLP_ENDPOINT to collector address

3. DEPLOY OpenTelemetry Collector (if not already running)
   - Local: docker compose -f docker-compose-otel-collector-contrib.yml up -d
   - Production: deploy collector with exporters for your backend

4. START FeatBit services with OTEL enabled
   - Local: docker compose -f docker-compose-otel.yml up -d
   - Production: restart services with updated env vars

5. VERIFY telemetry
   - Check collector logs for incoming data
   - Open backend UIs (Seq/Jaeger/Prometheus or equivalent)
   - Trigger feature flag operations and confirm metrics/traces/logs appear
```

## Instrumented Services

FeatBit has three backend services, each with OpenTelemetry automatic instrumentation:

| Service | Language | Instrumentation Docs |
|---------|----------|---------------------|
| Api Service | .NET/C# | [.NET Automatic Instrumentation](https://opentelemetry.io/docs/languages/net/automatic/) |
| Evaluation-Server | .NET/C# | [.NET Automatic Instrumentation](https://opentelemetry.io/docs/languages/net/automatic/) |
| Data Analytic Service | Python | [Python Automatic Instrumentation](https://opentelemetry.io/docs/languages/python/automatic/) |

Telemetry signals collected: metrics (CPU, memory, network), traces (request flows, latency), and logs (application events, errors).

## Environment Variables

Set these for each service to enable OpenTelemetry:

```bash
# Enable OpenTelemetry
ENABLE_OPENTELEMETRY=true

# Assign a unique name per service
OTEL_SERVICE_NAME=featbit-api           # Api Service
OTEL_SERVICE_NAME=featbit-els           # Evaluation-Server
OTEL_SERVICE_NAME=featbit-das           # Data Analytic Service

# Point to the OpenTelemetry Collector gRPC endpoint
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
```

For advanced configuration, refer to:
- .NET services: [Configuration docs](https://github.com/open-telemetry/opentelemetry-dotnet-instrumentation/blob/main/docs/config.md)
- Python service: [Environment variables](https://opentelemetry-python.readthedocs.io/en/latest/sdk/environment_variables.html)

## Local Development Example

Run the complete stack locally with Seq (logs), Jaeger (traces), and Prometheus (metrics):

```bash
git clone https://github.com/featbit/featbit.git
cd featbit

# Build test images
docker compose --project-directory . -f ./docker/composes/docker-compose-dev.yml build

# Start the OpenTelemetry Collector, Seq, Jaeger, and Prometheus
docker compose --project-directory . -f ./docker/composes/docker-compose-otel-collector-contrib.yml up -d

# Start FeatBit services with OpenTelemetry enabled
docker compose --project-directory . -f ./docker/composes/docker-compose-otel.yml up -d
```

After startup, trigger feature flag operations (create flags, evaluate, view insights), then verify telemetry at:
- **Seq (Logs)**: http://localhost:8082
- **Jaeger (Traces)**: http://localhost:16686
- **Prometheus (Metrics)**: http://localhost:9090

## Production Deployment

1. Assign a unique `OTEL_SERVICE_NAME` to each service to distinguish telemetry
2. Set exporter endpoints with correct protocol (http/https) and ports
3. Configure sampling rates to control telemetry data volume
4. Monitor all three services for complete visibility
5. Configure one of these backends (or any OpenTelemetry-compatible system):
   - Datadog
   - New Relic One
   - Grafana
   - Seq / Jaeger / Prometheus

## Monitoring and Troubleshooting

- Track request latency, throughput, and resource utilization via metrics
- Trace request flows to identify bottlenecks
- Correlate logs with traces to pinpoint root causes
- Set up alerts on performance anomalies in your backend

Official documentation: https://docs.featbit.co/integrations/observability/opentelemetry
