# Kubernetes Lab 12: DaemonSets

This lab focuses on node-level workloads, rollout behavior, and node targeting.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Basic DaemonSet
4. Lab 2: Node Selector
5. Lab 3: Tolerations for Control Plane
6. Lab 4: Rolling Update
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns ds-lab
kubectl config set-context --current --namespace=ds-lab
```

---

## 2. Core Concepts
- **DaemonSet:** One pod per node
- **Use cases:** Agents, log collectors, node monitors

---

## 3. Lab 1: Basic DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-echo
spec:
  selector:
    matchLabels:
      app: node-echo
  template:
    metadata:
      labels:
        app: node-echo
    spec:
      containers:
      - name: app
        image: busybox:1.36
        command: ["/bin/sh", "-c", "echo $NODE_NAME; sleep 3600"]
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
```

### Apply
```bash
kubectl apply -f - <<'YAML'
<PASTE THE YAML ABOVE>
YAML
kubectl get pods -o wide
```

---

## 4. Lab 2: Node Selector

```bash
kubectl label node <node-name> disk=ssd
```

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ssd-agent
spec:
  selector:
    matchLabels:
      app: ssd-agent
  template:
    metadata:
      labels:
        app: ssd-agent
    spec:
      nodeSelector:
        disk: ssd
      containers:
      - name: app
        image: busybox:1.36
        command: ["/bin/sh", "-c", "sleep 3600"]
```

---

## 5. Lab 3: Tolerations for Control Plane

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: cp-agent
spec:
  selector:
    matchLabels:
      app: cp-agent
  template:
    metadata:
      labels:
        app: cp-agent
    spec:
      tolerations:
      - key: "node-role.kubernetes.io/control-plane"
        operator: "Exists"
        effect: "NoSchedule"
      containers:
      - name: app
        image: busybox:1.36
        command: ["/bin/sh", "-c", "sleep 3600"]
```

---

## 6. Lab 4: Rolling Update

```bash
kubectl set image ds/node-echo app=busybox:1.36
kubectl rollout status ds/node-echo
```

---

## 7. Troubleshooting Scenarios
- **Missing pods:** Node selectors or taints block scheduling.
- **Update stuck:** Check pod readiness and node pressure.
- **Too many pods:** DaemonSet should match nodes count.

---

## 8. Interview Questions and Answers

### Q1: When use DaemonSet?
**Answer:** For node-level agents like logging or monitoring.

### Q2: How is DaemonSet different from Deployment?
**Answer:** DaemonSet runs one pod per node; Deployment scales by replica count.

### Q3: How do you target specific nodes?
**Answer:** Use nodeSelector, nodeAffinity, or taints/tolerations.

---

## 9. Cleanup
```bash
kubectl delete ns ds-lab
```

---

## 10. Theory (Concise)
- **DaemonSet maps pods to nodes**, not replicas.
- **Node selection controls placement** for infra agents.
- **Rollout behavior** is similar to Deployments.
