# Kubernetes Lab 14: Scheduling, Priority, and Preemption

This lab explores PriorityClass, preemption, and scheduling decisions.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Create PriorityClasses
4. Lab 2: Launch Low and High Priority Pods
5. Lab 3: Preemption Policy
6. Lab 4: Scheduling Debugging
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns priority-lab
kubectl config set-context --current --namespace=priority-lab
```

---

## 2. Core Concepts
- **PriorityClass:** Assigns relative importance
- **Preemption:** Evicts lower-priority pods when needed

---

## 3. Lab 1: Create PriorityClasses

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
preemptionPolicy: PreemptLowerPriority
globalDefault: false
description: "High priority"
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 1000
preemptionPolicy: PreemptLowerPriority
globalDefault: false
description: "Low priority"
```

---

## 4. Lab 2: Launch Low and High Priority Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: low-pod
spec:
  priorityClassName: low-priority
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 3600"]
---
apiVersion: v1
kind: Pod
metadata:
  name: high-pod
spec:
  priorityClassName: high-priority
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 3600"]
```

### Solution check
`kubectl describe pod` shows priority values.

---

## 5. Lab 3: Preemption Policy

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-preempt
spec:
  priorityClassName: high-priority
  preemptionPolicy: Never
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 3600"]
```

### Solution check
Pod will not preempt others even if resources are tight.

---

## 6. Lab 4: Scheduling Debugging

```bash
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## 7. Troubleshooting Scenarios
- **Pods Pending:** Check resource requests and node capacity.
- **No preemption:** Preemption policy or PDBs may block eviction.
- **Priority ignored:** Ensure PriorityClass exists and is referenced.

---

## 8. Interview Questions and Answers

### Q1: What triggers preemption?
**Answer:** A higher-priority pod cannot be scheduled due to lack of resources.

### Q2: What is globalDefault?
**Answer:** The default PriorityClass used if none is set on a pod.

### Q3: How do PDBs affect preemption?
**Answer:** They can prevent eviction to maintain availability.

---

## 9. Cleanup
```bash
kubectl delete ns priority-lab
```

---

## 10. Theory (Concise)
- **Priority influences scheduling order** and eviction decisions.
- **Preemption is a last resort** when resources are scarce.
- **Policies and PDBs** can override eviction behavior.
