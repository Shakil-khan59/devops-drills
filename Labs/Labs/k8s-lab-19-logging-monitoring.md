# Kubernetes Lab 19: Logging and Monitoring

This lab builds operational skills for logs, metrics, and basic observability.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Application Logs
4. Lab 2: Previous Logs and Crash Analysis
5. Lab 3: Metrics with kubectl top
6. Lab 4: Prometheus Annotations (Optional)
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
- Metrics Server for `kubectl top`

```bash
kubectl create ns observability-lab
kubectl config set-context --current --namespace=observability-lab
```

---

## 2. Core Concepts
- **Logs:** Per-container stdout/stderr
- **Metrics:** CPU/memory and custom metrics
- **Observability:** Logs, metrics, traces

---

## 3. Lab 1: Application Logs

```bash
kubectl apply -f - <<'YAML'
apiVersion: v1
kind: Pod
metadata:
  name: log-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "while true; do echo hello $(date); sleep 2; done"]
YAML

kubectl logs -f log-demo
```

---

## 4. Lab 2: Previous Logs and Crash Analysis

```bash
kubectl apply -f - <<'YAML'
apiVersion: v1
kind: Pod
metadata:
  name: crash-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "echo crash; exit 1"]
YAML

kubectl logs crash-demo --previous
```

---

## 5. Lab 3: Metrics with kubectl top

```bash
kubectl top pods
kubectl top nodes
```

---

## 6. Lab 4: Prometheus Annotations (Optional)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: metrics-demo
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9102"
spec:
  containers:
  - name: app
    image: nginx:1.25
```

---

## 7. Troubleshooting Scenarios
- **No logs:** Check container name and pod status.
- **No metrics:** Metrics server not installed or misconfigured.
- **High CPU:** Use `kubectl top` and resource limits.

---

## 8. Interview Questions and Answers

### Q1: Where do Kubernetes logs live?
**Answer:** Container stdout/stderr, collected by node agents.

### Q2: Why use metrics server?
**Answer:** Provides resource metrics for HPA and `kubectl top`.

### Q3: How do you trace a crash loop?
**Answer:** Use `kubectl logs --previous` and describe events.

---

## 9. Cleanup
```bash
kubectl delete ns observability-lab
```

---

## 10. Theory (Concise)
- **Logs are node-local** but often shipped to a backend.
- **Metrics enable autoscaling** and capacity planning.
- **Observability is multi-signal** by design.
