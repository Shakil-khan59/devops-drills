# Kubernetes Lab 29: Distributed Tracing (OpenTelemetry)

This lab introduces tracing concepts and an optional OpenTelemetry setup.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Tracing-Ready App (Concept)
4. Lab 2: OpenTelemetry Collector (Optional)
5. Lab 3: Export Traces to Jaeger (Optional)
6. Lab 4: Trace Sampling Strategy
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
- Optional: OpenTelemetry collector and Jaeger installed

```bash
kubectl create ns tracing-lab
kubectl config set-context --current --namespace=tracing-lab
```

---

## 2. Core Concepts
- **Traces:** End-to-end request timelines
- **Spans:** Units of work within a trace
- **Sampling:** Controls volume and cost

---

## 3. Lab 1: Tracing-Ready App (Concept)

> Use an app instrumented with OpenTelemetry SDK or sidecar auto-instrumentation.

---

## 4. Lab 2: OpenTelemetry Collector (Optional)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-config
data:
  otel-collector-config: |
    receivers:
      otlp:
        protocols:
          grpc:
    exporters:
      logging:
        loglevel: debug
    service:
      pipelines:
        traces:
          receivers: [otlp]
          exporters: [logging]
```

---

## 5. Lab 3: Export Traces to Jaeger (Optional)

> Deploy Jaeger and configure OTEL exporter to send traces.

---

## 6. Lab 4: Trace Sampling Strategy

### Practice
- Start with 1-5% sampling
- Increase temporarily during incidents

---

## 7. Troubleshooting Scenarios
- **No traces:** Check OTEL SDK config and collector endpoints.
- **High volume:** Tune sampling rate and filters.
- **Missing spans:** Ensure propagation headers are passed.

---

## 8. Interview Questions and Answers

### Q1: Why distributed tracing?
**Answer:** To debug latency and dependencies across services.

### Q2: What is a span?
**Answer:** A timed operation within a trace.

### Q3: How do you control trace cost?
**Answer:** Sampling and selective instrumentation.

---

## 9. Cleanup
```bash
kubectl delete ns tracing-lab
```

---

## 10. Theory (Concise)
- **Tracing reveals latency paths** across services.
- **Sampling balances signal vs cost**.
- **Propagation context is critical** for trace continuity.
