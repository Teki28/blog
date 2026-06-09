---
layout: "../../layouts/BlogPostLayout.astro"
title: Observability Part 2 - Practice With Spring Boot and OpenTelemetry
date: 2026-06-09
author: Teki
image: {
  src: "/images/post-16/cover.png",
  alt: "cover image",
}
description: This is the second part of a two-part series on observability. In Part 1, we covered the theory and concepts behind the three pillars of observability - metrics, traces, and logs. In this post, we will put that theory into practice by implementing an observability architecture for a sample Spring Boot application using OpenTelemetry. Github repo - https://github.com/Teki28/observability-springboot.
draft: false
category: Coding
---

# Observability Architecture

In a nutshell, applications emit three types of telemetry signals — **metrics**, **traces**, and **logs** to OTel Collector, which fans out to three backends — Prometheus for metrics, Tempo for traces, and Loki. This separates the concerns of metrics collection and prevents vendor lock-in, while still allowing for deep correlation between signals in Grafana.

This document describes how the three observability signals — **metrics**, **traces**, and **logs** — are collected, shipped, stored, and correlated across the three Spring Boot services in this project.

---

## Table of Contents

1. [Big Picture](#big-picture)
2. [Signal 1: Metrics](#signal-1-metrics)
3. [Signal 2: Traces](#signal-2-traces)
4. [Signal 3: Logs](#signal-3-logs)
5. [Cross-Service Trace Propagation](#cross-service-trace-propagation)
6. [Trace ↔ Log Correlation](#trace--log-correlation)
7. [Grafana: Navigating Between Signals](#grafana-navigating-between-signals)
8. [Configuration Reference](#configuration-reference)

---

## Big Picture

```
┌────────────────────────────────────────────────────────────────────────┐
│                         Spring Boot Services                           │
│                                                                        │
│  store-service :8080    bank-service :8081    stock-service :8082      │
│  ┌───────────────────────────────────────────────────────────────┐     │
│  │  Micrometer (metrics + tracing)  +  Logback (logs)            │     │
│  └─────────────┬──────────────────────────┬───────────────────┘  │     │
│                │ OTLP HTTP /v1/traces      │ OTLP HTTP /v1/logs   │     │
│                │ :4318                     │ :4318                │     │
│  /actuator/prometheus (pull)               │                      │     │
└────────────────┼───────────────────────────┼──────────────────────┘     │
                 │                           │                            │
                 ▼                           ▼                            │
        ┌─────────────────────────────────────────┐                      │
        │           OTel Collector  :4317/:4318    │                      │
        │  receivers: otlp (grpc + http)           │                      │
        │  processors: batch (1s timeout)          │                      │
        │  exporters: otlp/tempo, otlphttp/loki    │                      │
        └──────────────┬──────────────────┬────────┘                      │
                       │ gRPC             │ HTTP                          │
                       ▼                 ▼                               │
              ┌──────────────┐   ┌──────────────┐   ┌──────────────┐    │
              │    Tempo     │   │     Loki     │   │  Prometheus  │    │
              │  (traces)    │   │   (logs)     │   │  (metrics)   │◄───┘
              │    :3200     │   │    :3100     │   │    :9090     │ (scrape)
              └──────┬───────┘   └──────┬───────┘   └──────┬───────┘
                     │                  │                   │
                     └──────────────────┼───────────────────┘
                                        ▼
                              ┌──────────────────┐
                              │     Grafana      │
                              │      :3000       │
                              │  (dashboards +   │
                              │   correlations)  │
                              └──────────────────┘
```

Three distinct paths carry the three signals. Metrics take a **pull** path (Prometheus scrapes each service directly). Traces and logs both take a **push** path (services send OTLP to the collector, which fans out to Tempo and Loki).

---

## Signal 1: Metrics

### How It Works

Spring Boot Actuator auto-configures a Micrometer `MeterRegistry`. Each service:
1. Registers standard meters automatically: JVM memory, GC, HTTP server request durations, thread pools, etc.
2. Registers domain-specific meters explicitly in service constructors (see [Custom Metrics](#custom-metrics) below).

Prometheus scrapes each service's `/actuator/prometheus` endpoint every 15 seconds.

```
Service /actuator/prometheus  ──(HTTP GET, every 15s)──►  Prometheus  ──►  Grafana
```

### Prometheus Configuration

`prometheus.yml` lists three static scrape targets, one per service:

```yaml
scrape_configs:
  - job_name: store-service
    static_configs:
      - targets: ['store-service:8080']
    metrics_path: /actuator/prometheus
  # ... bank-service:8081, stock-service:8082
```

All metrics emitted by a service carry an `application` label (e.g. `application="store-service"`) set via:

```yaml
# application.yml (all services)
management:
  metrics:
    tags:
      application: ${spring.application.name}
```

### Custom Metrics

Each service's constructor pre-registers meters so that Prometheus sees them even before the first operation — avoiding gaps in time-series data.

**bank-service** (`BankService.java`):

| Metric | Type | Labels | Description |
|---|---|---|---|
| `bank.balance.cents` | Gauge | `accountId` | Live balance, updated atomically |
| `bank.debit.count` | Counter | `accountId` | Number of debit operations |
| `bank.debit.cents` | Counter | `accountId` | Total cents debited |
| `bank.credit.count` | Counter | `accountId` | Number of credit operations |
| `bank.credit.cents` | Counter | `accountId` | Total cents credited |

**stock-service** (`StockService.java`):

| Metric | Type | Labels | Description |
|---|---|---|---|
| `stock.inventory.quantity` | Gauge | `itemId` | Live stock level |
| `stock.add.count` | Counter | `itemId` | Number of add operations |
| `stock.add.units` | Counter | `itemId` | Total units added |
| `stock.remove.count` | Counter | `itemId` | Number of remove operations |
| `stock.remove.units` | Counter | `itemId` | Total units removed |

**store-service** (`StoreService.java`):

| Metric | Type | Labels | Description |
|---|---|---|---|
| `store.buy.count` | Counter | `itemId` | Successful buy operations |
| `store.buy.failure` | Counter | `itemId` | Failed buy operations |
| `store.sell.count` | Counter | `itemId` | Successful sell operations |
| `store.sell.failure` | Counter | `itemId` | Failed sell operations |
| `store.buy.duration` | Timer | `itemId` | End-to-end buy latency (bank + stock calls) |
| `store.sell.duration` | Timer | `itemId` | End-to-end sell latency |

The `Timer.Sample` pattern captures both the happy path and exception path because `sample.stop(...)` is called in `finally`.

---

## Signal 2: Traces

### Dependency Chain

Three libraries work together and each plays a distinct role:

```
Spring Web (MVC)
    │  emits Micrometer Observations for each HTTP request
    ▼
Micrometer Tracing + micrometer-tracing-bridge-otel
    │  translates Micrometer Observations into OTel Spans
    ▼
OpenTelemetry SDK  (auto-configured by Spring Boot)
    │  manages Tracer, SpanProcessor, SpanExporter
    ▼
opentelemetry-exporter-otlp
    │  serialises spans as OTLP protobuf over HTTP
    ▼
OTel Collector  :4318/v1/traces
    │  batches and forwards
    ▼
Tempo  (gRPC :4317)
```

Spring Boot's auto-configuration wires all of this from classpath presence alone — no `@Bean` definitions needed for the tracing pipeline itself.

### Sampling

All three services sample 100% of requests:

```yaml
management:
  tracing:
    sampling:
      probability: 1.0
```

### What Gets a Span Automatically

- Every inbound HTTP request handled by Spring MVC → one server span
- Every outbound `RestClient` call made by store-service → one client span (nested under the server span)

This means a single `POST /store/buy` call produces a trace with three spans:
1. `POST /store/buy` — store-service (root span)
2. `POST /bank/debit` — store-service acting as HTTP client (child)
3. `POST /stock/remove` — store-service acting as HTTP client (child)

The spans in bank-service and stock-service are linked as the remote continuations of spans 2 and 3 respectively.

### Tempo Storage

Tempo stores traces locally (filesystem, WAL + blocks). In this dev setup:

- Block retention: **1 hour** — traces are discarded after an hour
- Max block duration: **5 minutes** — blocks are flushed every 5 minutes

---

## Signal 3: Logs

### Dependency Chain

```
Application code calls SLF4J Logger
    │
    ▼
Logback  (logback-spring.xml)
    │
    ├──► ConsoleAppender  (stdout, human-readable)
    │
    └──► OpenTelemetryAppender  (opentelemetry-logback-appender-1.0)
              │  serialises log records as OTLP LogRecord protobuf over HTTP
              ▼
         OTel Collector  :4318/v1/logs
              │  batches and forwards
              ▼
         Loki  (HTTP /otlp)
```

### Wiring the OTel SDK Into Logback

The `OpenTelemetryAppender` declared in `logback-spring.xml` is a Logback appender, but it needs a live `OpenTelemetry` SDK instance to export records. That instance is created by Spring Boot's auto-configuration, which runs after Logback initialises. To bridge the timing gap, each service registers:

```java
// config/OpenTelemetryLogConfig.java (in all three services)
@Bean
SmartInitializingSingleton installOtelLogAppender(OpenTelemetry openTelemetry) {
    return () -> OpenTelemetryAppender.install(openTelemetry);
}
```

`SmartInitializingSingleton.afterSingletonsInstantiated()` fires after the entire Spring context is ready, so the OTel SDK is fully initialised before the appender is wired in.

### logback-spring.xml

```xml
<appender name="OpenTelemetry"
          class="io.opentelemetry.instrumentation.logback.appender.v1_0.OpenTelemetryAppender">
    <captureCodeAttributes>true</captureCodeAttributes>   <!-- class, method, line -->
    <captureMarkerAttribute>true</captureMarkerAttribute>
</appender>
```

`captureCodeAttributes` embeds the source file location into each log record, making it possible to jump from a Loki log entry back to the exact line of code.

### Trace ID in Console Logs

The console pattern adds the current trace and span IDs to every log line via MDC:

```yaml
# application.yml
logging:
  pattern:
    level: "%5p [${spring.application.name},%X{traceId:-},%X{spanId:-}]"
```

A log line looks like:

```
INFO [store-service,4bf92f3577b34da6a3ce929d0e0e4736,00f067aa0ba902b7] Processing buy: ...
```

The MDC values are populated automatically by the Micrometer tracing bridge whenever a span is active.

### Loki Storage

Loki receives log records via its OTLP HTTP endpoint (`/otlp`). It stores them using TSDB (schema v13) on the local filesystem. `allow_structured_metadata: true` means the structured key-value attributes sent by the OTel appender (e.g. `traceId`, code location) are stored as indexed metadata alongside the log body, making them queryable in LogQL.

---

## Cross-Service Trace Propagation

When store-service calls bank-service or stock-service, the active span's context must travel with the HTTP request so that the downstream service can create a child span in the same trace.

### How It Happens Automatically

Spring Boot's auto-configuration registers a `RestClient.Builder` as a prototype bean. The builder is pre-configured with Micrometer observation interceptors that:
1. Start a client-side span for the outbound call
2. Inject the current trace context as **W3C `traceparent`** and `tracestate` headers

```java
// store-service RestClientConfig.java
@Bean
public RestClient bankRestClient(
        RestClient.Builder builder,     // ← this is the auto-configured prototype
        @Value("${services.bank.url}") String bankUrl) {
    return builder.baseUrl(bankUrl).build();
}
```

No additional wiring is needed. If a plain `new RestClient(...)` were used instead, context propagation would silently break.

### What the Downstream Service Sees

bank-service and stock-service receive the `traceparent` header on every request from store-service. Spring MVC's instrumentation reads it, creates a server-side span as a child of the incoming span context, and the trace continues seamlessly.

### W3C TraceContext Header Format

```
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
             ^^ ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ ^^^^^^^^^^^^^^^^ ^^
             version       trace-id (128-bit)     parent-span-id   flags
```

---

## Trace ↔ Log Correlation

The same `traceId` appears in both Tempo and Loki. The two systems are linked so you can navigate between them with one click.

### How the traceId Gets Into Logs

Micrometer's tracing bridge writes the active `traceId` and `spanId` into SLF4J's MDC whenever a span is started. The OTel Logback appender reads MDC on every `Logger.info(...)` call and attaches the IDs as structured attributes on the OTLP `LogRecord` it ships to Loki.

This means every log line emitted during a request automatically carries the trace ID — no manual `MDC.put(...)` calls needed in application code.

### Loki → Tempo (log line to trace)

Grafana's Loki datasource is configured with a **derived field** that detects `traceId` in structured log attributes:

```yaml
# grafana/provisioning/datasources/datasources.yml
derivedFields:
  - datasourceUid: tempo
    matcherRegex: '"traceId":"(\w+)"'
    name: TraceID
    url: "$${__value.raw}"
    urlDisplayLabel: View trace in Tempo
```

In the Grafana Logs panel, each log line with a `traceId` value gets a **"View trace in Tempo"** button that opens the full distributed trace.

### Tempo → Loki (trace span to logs)

The Tempo datasource is configured with `tracesToLogsV2`:

```yaml
tracesToLogsV2:
  datasourceUid: loki
  filterByTraceID: true
  filterBySpanID: true
```

In the Grafana Tempo panel, clicking any span shows a **"Logs"** link that queries Loki for all log lines that share that span's `traceId` (and optionally `spanId`).

### Tempo → Service Map

The Tempo datasource also has:

```yaml
serviceMap:
  datasourceUid: prometheus
```

This enables Grafana's **Service Map** view, which reads Prometheus metrics to overlay request rates and error rates onto a visual topology of the services derived from trace data.

---

## Grafana: Navigating Between Signals

Grafana is the single pane of glass for all three signals. Datasources are provisioned automatically at startup from `grafana/provisioning/datasources/datasources.yml` — no manual setup needed.

| Datasource | UID | Backend |
|---|---|---|
| Prometheus | `prometheus` | `http://prometheus:9090` |
| Tempo | `tempo` | `http://tempo:3200` |
| Loki | `loki` | `http://loki:3100` |

Grafana runs with anonymous admin access (`GF_AUTH_ANONYMOUS_ENABLED=true`, `GF_AUTH_ANONYMOUS_ORG_ROLE=Admin`) so no login is required.

### Navigation Flows

```
Metrics (Prometheus)
    │  spot a spike in store.buy.failure
    │
    ▼
Traces (Tempo / Explore)
    │  filter by service + time window → find failing traces
    │  click a span → see "Logs" link
    │
    ▼
Logs (Loki)
    │  read the full log context around the failure
    │  click "View trace in Tempo" on any log line
    │
    ▼
Traces (Tempo) ← back to trace, or pivot to another span
```

---

## Configuration Reference

### application.yml (per service)

```yaml
management:
  tracing:
    sampling:
      probability: 1.0          # 100% trace sampling
  otlp:
    tracing:
      endpoint: http://localhost:4318/v1/traces   # overridden in Docker by env var
    logging:
      endpoint: http://localhost:4318/v1/logs     # overridden in Docker by env var
  metrics:
    tags:
      application: ${spring.application.name}    # adds 'application' label to all metrics

logging:
  pattern:
    level: "%5p [${spring.application.name},%X{traceId:-},%X{spanId:-}]"
```

### Docker Compose Overrides

When running inside Docker, the OTLP endpoint env vars point at the collector container instead of localhost:

```yaml
environment:
  MANAGEMENT_OTLP_TRACING_ENDPOINT: http://otel-collector:4318/v1/traces
  MANAGEMENT_OTLP_LOGGING_ENDPOINT: http://otel-collector:4318/v1/logs
```

store-service also gets its upstream service URLs overridden:

```yaml
environment:
  SERVICES_BANK_URL:  http://bank-service:8081
  SERVICES_STOCK_URL: http://stock-service:8082
```

### OTel Collector Pipelines

```
Traces pipeline:  otlp receiver → batch processor → otlp/tempo exporter (gRPC)
Logs pipeline:    otlp receiver → batch processor → otlphttp/loki exporter (HTTP)
```

The collector also runs a `debug` exporter on both pipelines, writing a summary of each batch to its stdout — useful for verifying telemetry is flowing without querying Grafana.