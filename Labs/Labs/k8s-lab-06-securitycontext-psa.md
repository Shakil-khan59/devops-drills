# Kubernetes Lab 06: SecurityContext and Pod Security Admission

This lab focuses on pod hardening, privilege reduction, and namespace policies.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Run as Non-Root
4. Lab 2: Read-Only Root Filesystem
5. Lab 3: Drop Linux Capabilities
6. Lab 4: Pod Security Admission (Baseline/Restricted)
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns security-lab
kubectl config set-context --current --namespace=security-lab
```

---

## 2. Core Concepts
- **SecurityContext:** Per-pod or per-container security settings
- **PSA:** Namespace-level policy controls (Baseline/Restricted)

---

## 3. Lab 1: Run as Non-Root

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nonroot-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "id; sleep 3600"]
    securityContext:
      runAsNonRoot: true
      runAsUser: 1000
```

### Apply
```bash
kubectl apply -f - <<'YAML'
<PASTE THE YAML ABOVE>
YAML
```

### Solution check
`kubectl logs nonroot-demo` shows UID 1000.

---

## 4. Lab 2: Read-Only Root Filesystem

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "echo test > /tmp/x; sleep 3600"]
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp
    emptyDir: {}
```

### Solution check
Write succeeds only on `/tmp` (mounted volume).

---

## 5. Lab 3: Drop Linux Capabilities

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: caps-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "capsh --print; sleep 3600"]
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
```

### Solution check
Capabilities list is empty.

---

## 6. Lab 4: Pod Security Admission (Baseline/Restricted)

### Apply PSA labels
```bash
kubectl label ns security-lab pod-security.kubernetes.io/enforce=restricted
kubectl label ns security-lab pod-security.kubernetes.io/warn=restricted
```

### Try a privileged pod (should be blocked)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privileged-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 3600"]
    securityContext:
      privileged: true
```

### Solution check
Admission is denied under `restricted`.

---

## 7. Troubleshooting Scenarios
- **Admission denied:** Check PSA labels and workload security context.
- **Permission errors:** Verify volumes are writable or add emptyDir.
- **Capabilities missing:** Some tools require NET_ADMIN or SYS_TIME.

---

## 8. Interview Questions and Answers

### Q1: Why enforce non-root containers?
**Answer:** It reduces blast radius if a container is compromised.

### Q2: What is PSA?
**Answer:** A built-in admission policy enforcing baseline/restricted controls per namespace.

### Q3: How do you make a container write to /tmp with readOnlyRootFilesystem?
**Answer:** Mount an emptyDir at /tmp.

---

## 9. Cleanup
```bash
kubectl delete ns security-lab
```

---

## 10. Theory (Concise)
- **SecurityContext reduces privileges** at pod/container level.
- **PSA enforces policy** at namespace scope.
- **Least privilege** is a core production baseline.
