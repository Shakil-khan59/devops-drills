# Kubernetes Lab 08: Resource Requests, Limits, and QoS

This lab covers resource management, QoS classes, and capacity planning signals.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: BestEffort QoS
4. Lab 2: Burstable QoS
5. Lab 3: Guaranteed QoS
6. Lab 4: LimitRange Defaults
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns qos-lab
kubectl config set-context --current --namespace=qos-lab
```

---

## 2. Core Concepts
- **Requests:** Scheduling guarantee
- **Limits:** Runtime cap
- **QoS:** BestEffort, Burstable, Guaranteed

---

## 3. Lab 1: BestEffort QoS

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: besteffort
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 3600"]
```

### Apply and verify
```bash
kubectl apply -f - <<'YAML'
<PASTE THE YAML ABOVE>
YAML
kubectl describe pod besteffort | rg -n "QoS"
```

---

## 4. Lab 2: Burstable QoS

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: burstable
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 3600"]
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "500m"
        memory: "256Mi"
```

---

## 5. Lab 3: Guaranteed QoS

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guaranteed
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 3600"]
    resources:
      requests:
        cpu: "200m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "128Mi"
```

---

## 6. Lab 4: LimitRange Defaults

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
spec:
  limits:
  - type: Container
    defaultRequest:
      cpu: "100m"
      memory: "64Mi"
    default:
      cpu: "500m"
      memory: "256Mi"
```

### Apply
```bash
kubectl apply -f - <<'YAML'
<PASTE THE YAML ABOVE>
YAML
```

### Solution check
New pods inherit default requests/limits if not set.

---

## 7. Troubleshooting Scenarios
- **Pod Pending:** Requests exceed node capacity.
- **OOMKilled:** Container exceeded memory limit.
- **Throttling:** CPU limit too low for workload.

---

## 8. Interview Questions and Answers

### Q1: Requests vs limits?
**Answer:** Requests affect scheduling; limits constrain runtime usage.

### Q2: QoS classes?
**Answer:** BestEffort (none), Burstable (requests != limits), Guaranteed (requests = limits).

### Q3: Why use LimitRange?
**Answer:** To enforce defaults and avoid unbounded usage.

---

## 9. Cleanup
```bash
kubectl delete ns qos-lab
```

---

## 10. Theory (Concise)
- **Requests drive placement**, limits protect nodes.
- **QoS affects eviction priority** under pressure.
- **Defaults reduce risk** in shared clusters.
