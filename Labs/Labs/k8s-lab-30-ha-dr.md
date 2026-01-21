# Kubernetes Lab 30: High Availability and Disaster Recovery

This lab covers HA concepts, etcd backups, and recovery planning.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Validate Control Plane HA (Concept)
4. Lab 2: etcd Snapshot (If Accessible)
5. Lab 3: Node Failure Simulation
6. Lab 4: Recovery Checklist
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
- Access to control plane nodes may be required for etcd snapshots

---

## 2. Core Concepts
- **HA control plane:** Multiple API servers and etcd nodes
- **DR:** Backup, restore, and recovery procedures

---

## 3. Lab 1: Validate Control Plane HA (Concept)

```bash
kubectl get nodes
kubectl get pods -n kube-system | rg -n "apiserver|etcd"
```

---

## 4. Lab 2: etcd Snapshot (If Accessible)

```bash
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd.snapshot \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

---

## 5. Lab 3: Node Failure Simulation

```bash
kubectl cordon <node-name>
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

### Solution check
Workloads reschedule and remain available.

---

## 6. Lab 4: Recovery Checklist

### Practice
- Verify backups exist
- Validate restore steps on a staging cluster
- Document RTO/RPO targets

---

## 7. Troubleshooting Scenarios
- **etcd snapshot fails:** Check certs and endpoint access.
- **API down:** Ensure control plane quorum.
- **Data loss:** Validate backup cadence and restore tests.

---

## 8. Interview Questions and Answers

### Q1: What is etcd?
**Answer:** The key-value store backing Kubernetes state.

### Q2: How do you ensure HA?
**Answer:** Multiple control plane nodes, etcd quorum, and load-balanced API.

### Q3: Why test restores?
**Answer:** Backups are only valuable if restores work.

---

## 9. Cleanup
```bash
# No namespace created; cleanup not required
```

---

## 10. Theory (Concise)
- **HA reduces downtime**, DR reduces data loss.
- **etcd is the source of truth**, protect it.
- **Practice restores** to ensure real recovery capability.
