# Kubernetes Lab 23: Namespaces, ResourceQuota, and LimitRange

This lab focuses on multi-tenant controls and resource governance.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Namespace Isolation
4. Lab 2: ResourceQuota
5. Lab 3: LimitRange Defaults
6. Lab 4: Enforcement and Errors
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns quota-lab
kubectl config set-context --current --namespace=quota-lab
```

---

## 2. Core Concepts
- **Namespaces:** Logical isolation
- **ResourceQuota:** Limits total resource usage
- **LimitRange:** Default per-container limits

---

## 3. Lab 1: Namespace Isolation

```bash
kubectl create ns team-a
kubectl create ns team-b
```

### Solution check
Resources in one namespace are not visible in another.

---

## 4. Lab 2: ResourceQuota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    pods: "4"
```

### Apply
```bash
kubectl apply -f - <<'YAML'
<PASTE THE YAML ABOVE>
YAML
```

---

## 5. Lab 3: LimitRange Defaults

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr
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

---

## 6. Lab 4: Enforcement and Errors

### Try to create too many pods
```bash
kubectl run p1 --image=busybox:1.36 --restart=Never -- sleep 3600
```

### Solution check
Creation fails once quota is exceeded.

---

## 7. Troubleshooting Scenarios
- **Quota exceeded:** Adjust hard limits or cleanup.
- **Unexpected defaults:** LimitRange applies to new pods only.
- **Namespace confusion:** Ensure context is correct.

---

## 8. Interview Questions and Answers

### Q1: Why use ResourceQuota?
**Answer:** To prevent noisy neighbor issues in shared clusters.

### Q2: How does LimitRange differ from Quota?
**Answer:** LimitRange sets per-container defaults; Quota caps totals.

### Q3: Can quotas be set per namespace?
**Answer:** Yes, they are namespace-scoped objects.

---

## 9. Cleanup
```bash
kubectl delete ns quota-lab team-a team-b
```

---

## 10. Theory (Concise)
- **Namespaces isolate resources**, not nodes.
- **Quotas control total consumption** per namespace.
- **LimitRanges enforce defaults** to reduce risk.
