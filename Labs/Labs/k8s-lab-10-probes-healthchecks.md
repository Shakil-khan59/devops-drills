# Kubernetes Lab 10: Probes and Health Checks

This lab builds skill in readiness, liveness, and startup probes.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Readiness Probe
4. Lab 2: Liveness Probe
5. Lab 3: Startup Probe
6. Lab 4: Probe Tuning and Failure Modes
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns probes-lab
kubectl config set-context --current --namespace=probes-lab
```

---

## 2. Core Concepts
- **Readiness:** Controls service traffic eligibility
- **Liveness:** Restarts unhealthy containers
- **Startup:** Delays liveness while app boots

---

## 3. Lab 1: Readiness Probe

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 10; touch /tmp/ready; httpd -f -p 8080"]
    readinessProbe:
      exec:
        command: ["/bin/sh", "-c", "test -f /tmp/ready"]
      periodSeconds: 5
```

### Solution check
Pod is not Ready for ~10 seconds, then becomes Ready.

---

## 4. Lab 2: Liveness Probe

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "httpd -f -p 8080; sleep 3600"]
    livenessProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
```

---

## 5. Lab 3: Startup Probe

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 20; httpd -f -p 8080"]
    startupProbe:
      httpGet:
        path: /
        port: 8080
      failureThreshold: 10
      periodSeconds: 3
    livenessProbe:
      httpGet:
        path: /
        port: 8080
      periodSeconds: 10
```

---

## 6. Lab 4: Probe Tuning and Failure Modes

### Practice
- Reduce `failureThreshold` and see restarts.
- Increase `periodSeconds` to reduce probe load.

---

## 7. Troubleshooting Scenarios
- **Pod never ready:** Readiness probe command failing.
- **Crash loops:** Liveness probe too aggressive.
- **Slow start:** Add startup probe to avoid early restarts.

---

## 8. Interview Questions and Answers

### Q1: Why not use only liveness?
**Answer:** Readiness controls traffic; liveness controls restarts.

### Q2: When use startupProbe?
**Answer:** For slow-starting apps that need extra time before liveness kicks in.

### Q3: How do probes affect rollouts?
**Answer:** Readiness gates rollout progress and service traffic.

---

## 9. Cleanup
```bash
kubectl delete ns probes-lab
```

---

## 10. Theory (Concise)
- **Readiness gates traffic**, liveness repairs failures.
- **Startup probes protect boot time**, preventing premature restarts.
- **Probe tuning** balances availability and resource cost.
