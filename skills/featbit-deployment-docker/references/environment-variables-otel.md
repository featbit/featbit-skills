---
name: environment-variables-otel
description: OpenTelemetry configuration variables for FeatBit Docker Compose deployments
---

# OpenTelemetry Environment Variables

FeatBit's backend services (Api, Evaluation-Server, and Data Analytic service) are instrumented with OpenTelemetry to publish observability metrics, traces, and logs.

## Table of Contents

1. [Basic OpenTelemetry Variables](#basic-opentelemetry-variables)
2. [Recommended Service Names](#recommended-service-names)
3. [Additional OpenTelemetry Variables](#additional-opentelemetry-variables)
4. [Complete OpenTelemetry Example](#complete-opentelemetry-example)
5. [Integration with Popular Observability Platforms](#integration-with-popular-observability-platforms)
6. [Testing OpenTelemetry Setup](#testing-opentelemetry-setup)
7. [Troubleshooting OpenTelemetry](#troubleshooting-opentelemetry)
8. [References](#references)

## Basic OpenTelemetry Variables

**Required variables** to enable OpenTelemetry for any service:

| Variable | Description | Example |
|----------|-------------|---------|
| `ENABLE_OPENTELEMETRY` | Enable/disable OpenTelemetry | `true` or `false` |
| `OTEL_SERVICE_NAME` | Service identifier in observability backends | `featbit-api`, `featbit-els`, `featbit-das` |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | gRPC endpoint of OpenTelemetry exporter | `http://otel-collector:4317` |

**Examples**:

```yaml
# API Server with OpenTelemetry
api-server:
  environment:
    - ENABLE_OPENTELEMETRY=true
    - OTEL_SERVICE_NAME=featbit-api
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317

# Evaluation Server with OpenTelemetry
evaluation-server:
  environment:
    - ENABLE_OPENTELEMETRY=true
    - OTEL_SERVICE_NAME=featbit-els
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317

# Data Analytics Server with OpenTelemetry
da-server:
  environment:
    - ENABLE_OPENTELEMETRY=true
    - OTEL_SERVICE_NAME=featbit-das
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
```

## Recommended Service Names

Use these consistent service names for easier identification:

- **API Server**: `featbit-api`
- **Evaluation Server**: `featbit-els`
- **Data Analytics Server**: `featbit-das`

## Additional OpenTelemetry Variables

**API Server and Evaluation Server (.NET/C# services)**:

These services support [.NET Automatic Instrumentation environment variables](https://github.com/open-telemetry/opentelemetry-dotnet-instrumentation/blob/main/docs/config.md).

Common additional variables:
```yaml
api-server:
  environment:
    - ENABLE_OPENTELEMETRY=true
    - OTEL_SERVICE_NAME=featbit-api
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
    # Additional configuration
    - OTEL_EXPORTER_OTLP_PROTOCOL=grpc  # or http/protobuf
    - OTEL_TRACES_SAMPLER=always_on     # Sampling strategy
    - OTEL_METRICS_EXPORTER=otlp        # Metrics exporter
    - OTEL_LOGS_EXPORTER=otlp           # Logs exporter
```

**Data Analytics Server (Python service)**:

This service supports [Python Automatic Instrumentation environment variables](https://opentelemetry-python.readthedocs.io/en/latest/sdk/environment_variables.html).

Common additional variables:
```yaml
da-server:
  environment:
    - ENABLE_OPENTELEMETRY=true
    - OTEL_SERVICE_NAME=featbit-das
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
    # Additional configuration
    - OTEL_EXPORTER_OTLP_PROTOCOL=grpc
    - OTEL_PYTHON_LOGGING_AUTO_INSTRUMENTATION_ENABLED=true
```

## Complete OpenTelemetry Example

Example configuration with Seq (logs), Jaeger (traces), and Prometheus (metrics):

```yaml
version: '3'

services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    command: ["--config=/etc/otel-collector-config.yaml"]
    volumes:
      - ./otel-collector-config.yaml:/etc/otel-collector-config.yaml
    ports:
      - "4317:4317"   # OTLP gRPC receiver
      - "4318:4318"   # OTLP HTTP receiver

  api-server:
    image: featbit/featbit-api-server:latest
    environment:
      # ... other variables ...
      - ENABLE_OPENTELEMETRY=true
      - OTEL_SERVICE_NAME=featbit-api
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
    depends_on:
      - otel-collector

  evaluation-server:
    image: featbit/featbit-evaluation-server:latest
    environment:
      # ... other variables ...
      - ENABLE_OPENTELEMETRY=true
      - OTEL_SERVICE_NAME=featbit-els
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
    depends_on:
      - otel-collector

  da-server:
    image: featbit/featbit-data-analytics-server:latest
    environment:
      # ... other variables ...
      - ENABLE_OPENTELEMETRY=true
      - OTEL_SERVICE_NAME=featbit-das
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
    depends_on:
      - otel-collector

  # Observability backends
  seq:
    image: datalust/seq:latest
    environment:
      - ACCEPT_EULA=Y
    ports:
      - "8082:80"     # Seq UI

  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686" # Jaeger UI

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"   # Prometheus UI
```

## Integration with Popular Observability Platforms

**Important**: FeatBit services only need the basic OpenTelemetry variables (`ENABLE_OPENTELEMETRY`, `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_ENDPOINT`). Integration with specific observability platforms is configured in the **OpenTelemetry Collector**, not in FeatBit services.

**Architecture**:
```
FeatBit Services → OpenTelemetry Collector → Observability Platform
                   (configure endpoint)    (configure platform credentials)
```

**FeatBit Service Configuration** (same for all platforms):
```yaml
api-server:
  environment:
    - ENABLE_OPENTELEMETRY=true
    - OTEL_SERVICE_NAME=featbit-api
    - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
```

**Platform Integration** is done in OpenTelemetry Collector configuration file:

**Example: Datadog Integration**

1. FeatBit services send to collector (configuration above)
2. Configure collector to forward to Datadog:

```yaml
# otel-collector-config.yaml
exporters:
  datadog:
    api:
      key: ${DD_API_KEY}
      site: datadoghq.com

service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [datadog]
    metrics:
      receivers: [otlp]
      exporters: [datadog]
```

**Example: Grafana Cloud Integration**

```yaml
# otel-collector-config.yaml
exporters:
  otlphttp:
    endpoint: https://otlp-gateway-prod-us-central-0.grafana.net/otlp
    headers:
      authorization: Basic ${GRAFANA_CLOUD_API_KEY}

service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [otlphttp]
```

**For detailed platform integration**, refer to:
- **Datadog**: https://docs.featbit.co/integrations/observability/datadog
- **New Relic**: https://docs.featbit.co/integrations/observability/newrelic
- **Grafana**: https://docs.featbit.co/integrations/observability/grafana
- **OpenTelemetry Collector Configuration**: https://opentelemetry.io/docs/collector/configuration/

## Testing OpenTelemetry Setup

After enabling OpenTelemetry, verify it's working:

1. **Check service logs**:
```bash
docker compose logs api-server | grep -i "opentelemetry"
docker compose logs api-server | grep -i "otel"
```

2. **Verify collector is receiving data**:
```bash
docker compose logs otel-collector
```

3. **Access observability backends**:
- Seq (logs): http://localhost:8082
- Jaeger (traces): http://localhost:16686
- Prometheus (metrics): http://localhost:9090

## Troubleshooting OpenTelemetry

**Issue**: No telemetry data appearing

**Solutions**:
1. Verify `ENABLE_OPENTELEMETRY=true` is set
2. Check collector endpoint is accessible from service
3. Check service logs for OpenTelemetry errors
4. Verify collector configuration is correct

**Issue**: Partial data (only traces, no metrics/logs)

**Solutions**:
1. Check exporter configuration for each signal type
2. Verify collector is configured to receive all signal types
3. Check backend supports all signal types

## References

- **FeatBit OpenTelemetry Documentation**: https://docs.featbit.co/integrations/observability/opentelemetry
- **.NET Auto Instrumentation Config**: https://github.com/open-telemetry/opentelemetry-dotnet-instrumentation/blob/main/docs/config.md
- **Python Auto Instrumentation Config**: https://opentelemetry-python.readthedocs.io/en/latest/sdk/environment_variables.html
- **OpenTelemetry Collector**: https://opentelemetry.io/docs/collector/
