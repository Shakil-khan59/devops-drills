# Kubernetes Lab 13: Affinity, Anti-Affinity, Taints, and Tolerations

This lab builds scheduling mastery with topology-aware placement.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Node Affinity
4. Lab 2: Pod Anti-Affinity
5. Lab 3: Taints and Tolerations
6. Lab 4: Topology Spread Constraints
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns affinity-lab
kubectl config set-context --current --namespace=affinity-lab
```

---

## 2. Core Concepts
- **Node affinity:** Prefer or require nodes with labels
- **Pod anti-affinity:** Spread replicas across topology
- **Taints:** Repel pods unless tolerated

---

## 3. Lab 1: Node Affinity

```bash
kubectl label node <node-name> zone=blue
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: zone
            operator: In
            values: ["blue"]
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 3600"]
```

---

## 4. Lab 2: Pod Anti-Affinity

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spread-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spread
  template:
    metadata:
      labels:
        app: spread
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: spread
              topologyKey: kubernetes.io/hostname
      containers:
      - name: app
        image: nginx:1.25
```

---

## 5. Lab 3: Taints and Tolerations

```bash
kubectl taint node <node-name> dedicated=lab:NoSchedule
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerate-demo
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "lab"
    effect: "NoSchedule"
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 3600"]
```

---

## 6. Lab 4: Topology Spread Constraints

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: topology-spread
spec:
  replicas: 4
  selector:
    matchLabels:
      app: topo
  template:
    metadata:
      labels:
        app: topo
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            app: topo
      containers:
      - name: app
        image: nginx:1.25
```

---

## 7. Troubleshooting Scenarios
- **Pods Pending:** Affinity constraints too strict.
- **Unexpected co-location:** Anti-affinity is preferred, not required.
- **Taints blocking:** Add correct tolerations.

---

## 8. Interview Questions and Answers

### Q1: Node affinity vs nodeSelector?
**Answer:** Node affinity is more expressive with operators and weights.

### Q2: What is a taint?
**Answer:** A node-level repel rule requiring tolerations.

### Q3: Why use topology spread constraints?
**Answer:** To reduce blast radius by spreading replicas.

---

## 9. Cleanup
```bash
kubectl delete ns affinity-lab
```

---

## 10. Theory (Concise)
- **Affinity controls placement**, not replica count.
- **Taints protect nodes** from unwanted pods.
- **Spread constraints improve resilience** across nodes/zones.
