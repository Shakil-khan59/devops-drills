# Kubernetes Lab 26: Node Maintenance and Upgrades

This lab covers safe node maintenance with cordon, drain, and eviction control.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Cordon and Uncordon
4. Lab 2: Drain a Node Safely
5. Lab 3: Pod Disruption Budgets
6. Lab 4: Maintenance Playbook
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns maintenance-lab
kubectl config set-context --current --namespace=maintenance-lab
```

---

## 2. Core Concepts
- **Cordon:** Prevent new scheduling
- **Drain:** Evict pods for maintenance
- **PDB:** Controls voluntary disruption

---

## 3. Lab 1: Cordon and Uncordon

```bash
kubectl get nodes
kubectl cordon <node-name>
kubectl uncordon <node-name>
```

---

## 4. Lab 2: Drain a Node Safely

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

### Solution check
Pods reschedule to other nodes.

---

## 5. Lab 3: Pod Disruption Budgets

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
```

### Solution check
Drain respects PDB and may block eviction.

---

## 6. Lab 4: Maintenance Playbook

### Practice
1. Cordon node
2. Drain node
3. Perform maintenance
4. Uncordon node

---

## 7. Troubleshooting Scenarios
- **Drain stuck:** PDB or Pod with local storage blocks eviction.
- **DaemonSet pods:** Use `--ignore-daemonsets`.
- **Eviction denied:** Check PDB settings.

---

## 8. Interview Questions and Answers

### Q1: Cordon vs drain?
**Answer:** Cordon prevents scheduling; drain evicts existing pods.

### Q2: Why do PDBs matter?
**Answer:** They preserve availability during disruptions.

### Q3: How to handle DaemonSets during drain?
**Answer:** Use `--ignore-daemonsets` and verify daemon health after.

---

## 9. Cleanup
```bash
kubectl delete ns maintenance-lab
```

---

## 10. Theory (Concise)
- **Maintenance requires controlled evictions** to avoid downtime.
- **PDBs enforce availability** during node work.
- **Drain workflows are standard** for safe upgrades.
