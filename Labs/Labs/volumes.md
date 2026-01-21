# Kubernetes Volumes Lab Tutorial (Mid/Senior Level)

This guide is a complete, hands-on path to mastering Kubernetes volumes. It is designed for mid-to-senior engineers and covers architecture, storage primitives, patterns, security, performance, troubleshooting, and interview scenarios.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts and Mental Models
3. Lab 1: Ephemeral Volumes (emptyDir)
4. Lab 2: Config and Secret Volumes
5. Lab 3: hostPath (When and Why Not)
6. Lab 4: PersistentVolume + PersistentVolumeClaim
7. Lab 5: StorageClass and Dynamic Provisioning
8. Lab 6: StatefulSet Volume Patterns
9. Lab 7: Access Modes and Multi-Attach Scenarios
10. Lab 8: Volume Expansion
11. Lab 9: Snapshots and Restore (If Supported)
12. Lab 10: Performance and I/O Observability
13. Troubleshooting Scenarios
14. Interview Questions and Answers
15. Cleanup
16. Theory (Concise)

---

## 1. Prerequisites and Setup

### Requirements
- Kubernetes cluster (kind/minikube/k3d/cloud)
- `kubectl`
- A default StorageClass (for dynamic provisioning)

### Verify cluster and StorageClass
```bash
kubectl cluster-info
kubectl get nodes
kubectl get sc
```

### Create a namespace
```bash
kubectl create ns volume-lab
kubectl config set-context --current --namespace=volume-lab
```

---

## 2. Core Concepts and Mental Models

### Key storage objects
- **Volume:** A directory with lifecycle tied to a pod (or node)
- **PersistentVolume (PV):** Cluster resource representing actual storage
- **PersistentVolumeClaim (PVC):** User request for storage
- **StorageClass (SC):** Defines how storage is provisioned dynamically

### Storage flow
1. Pod requests a PVC
2. PVC is bound to a PV (static or dynamic)
3. Pod mounts the PV through the PVC

### Why it matters for senior engineers
- Data durability guarantees
- Stateful workloads with zero data loss on reschedule
- Multi-AZ/region constraints and latency tradeoffs
- Security boundaries (secrets, encryption, access)

---

## 3. Lab 1: Ephemeral Volumes (emptyDir)

### Goal
Understand pod-scoped ephemeral storage.

### Manifest
Save as `lab1-emptydir.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "echo hello > /data/hello.txt; sleep 3600"]
    volumeMounts:
    - name: cache
      mountPath: /data
  volumes:
  - name: cache
    emptyDir: {}
```

### Apply and verify
```bash
kubectl apply -f lab1-emptydir.yaml
kubectl exec -it emptydir-demo -- cat /data/hello.txt
```

### Insight
Deleting the pod deletes the data.

---

## 4. Lab 2: Config and Secret Volumes

### Goal
Mount config and secrets as files.

### Create ConfigMap and Secret
```bash
kubectl create configmap app-config --from-literal=MODE=prod
kubectl create secret generic app-secret --from-literal=API_KEY=abc123
```

### Manifest
Save as `lab2-config-secret.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-secret-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "ls /etc/config /etc/secret; sleep 3600"]
    volumeMounts:
    - name: config
      mountPath: /etc/config
    - name: secret
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: config
    configMap:
      name: app-config
  - name: secret
    secret:
      secretName: app-secret
```

### Apply
```bash
kubectl apply -f lab2-config-secret.yaml
kubectl exec -it config-secret-demo -- sh -c 'cat /etc/config/MODE && cat /etc/secret/API_KEY'
```

---

## 5. Lab 3: hostPath (When and Why Not)

### Goal
Understand node-local storage and its risks.

### Manifest
Save as `lab3-hostpath.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "echo node-data > /host/data.txt; sleep 3600"]
    volumeMounts:
    - name: host
      mountPath: /host
  volumes:
  - name: host
    hostPath:
      path: /tmp/hostpath-demo
      type: DirectoryOrCreate
```

### Risk
If the pod is rescheduled to another node, data is lost.

---

## 6. Lab 4: PersistentVolume + PersistentVolumeClaim

### Goal
Use static provisioning (typical in on-prem clusters).

### Manifest
Save as `lab4-pv-pvc.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-lab
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /tmp/pv-lab
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-lab
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pv-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "echo persistent > /data/msg.txt; sleep 3600"]
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: pvc-lab
```

### Apply and verify
```bash
kubectl apply -f lab4-pv-pvc.yaml
kubectl exec -it pv-demo -- cat /data/msg.txt
```

---

## 7. Lab 5: StorageClass and Dynamic Provisioning

### Goal
Use a StorageClass to auto-provision volumes.

### Inspect StorageClasses
```bash
kubectl get sc
```

### Manifest
Save as `lab5-dynamic-pvc.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### Apply and verify
```bash
kubectl apply -f lab5-dynamic-pvc.yaml
kubectl get pvc
kubectl get pv
```

---

## 8. Lab 6: StatefulSet Volume Patterns

### Goal
Attach per-replica persistent volumes.

### Manifest
Save as `lab6-statefulset.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: st
spec:
  clusterIP: None
  selector:
    app: st
  ports:
  - port: 80
    name: http
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: st
spec:
  serviceName: st
  replicas: 2
  selector:
    matchLabels:
      app: st
  template:
    metadata:
      labels:
        app: st
    spec:
      containers:
      - name: app
        image: nginx:1.25
        volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

### Apply and verify
```bash
kubectl apply -f lab6-statefulset.yaml
kubectl get pvc
```

---

## 9. Lab 7: Access Modes and Multi-Attach Scenarios

### Goal
Understand RWO, ROX, RWX, and how they affect scheduling.

### Quick reference
- **ReadWriteOnce (RWO):** mounted read-write by one node
- **ReadOnlyMany (ROX):** mounted read-only by many nodes
- **ReadWriteMany (RWX):** mounted read-write by many nodes

### Scenario exercise
1. Create a PVC with RWO.
2. Schedule two pods on different nodes to mount the same PVC.
3. Observe one pod stuck in `Pending` due to multi-attach limitation.

### Example manifest
Save as `lab7-rwo-multiattach.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rwo-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: rwo-pod-a
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: rwo-pvc
---
apiVersion: v1
kind: Pod
metadata:
  name: rwo-pod-b
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: rwo-pvc
```

---

## 10. Lab 8: Volume Expansion

### Goal
Expand PVC size if StorageClass allows it.

### Check if expansion is allowed
```bash
kubectl get sc -o yaml | rg -n "allowVolumeExpansion"
```

### Expand PVC
```bash
kubectl patch pvc dynamic-pvc -p '{"spec":{"resources":{"requests":{"storage":"2Gi"}}}}'
```

### Verify
```bash
kubectl get pvc dynamic-pvc
```

---

## 11. Lab 9: Snapshots and Restore (If Supported)

> Some clusters require the `VolumeSnapshot` CRDs and a snapshot controller.

### Check snapshot support
```bash
kubectl get crd | rg -n "volumesnapshot"
```

### Example (if supported)
Save as `lab9-snapshot.yaml`:
```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: pvc-snap
spec:
  volumeSnapshotClassName: <YOUR_SNAPSHOT_CLASS>
  source:
    persistentVolumeClaimName: dynamic-pvc
```

### Restore (example)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restored-pvc
spec:
  dataSource:
    name: pvc-snap
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

## 12. Lab 10: Performance and I/O Observability

### Goal
Baseline I/O and validate volume behavior.

### Simple write test
```bash
kubectl run -it --rm fio --image=alpine:3.19 --restart=Never -- sh -c "apk add --no-cache fio; fio --name=test --directory=/data --size=50M --rw=write --bs=4k --numjobs=1"
```

> For production: use `fio` in a controlled environment and capture metrics via node exporter or CSI metrics.

---

## 13. Troubleshooting Scenarios

### Scenario A: PVC stuck in Pending
- Check StorageClass: `kubectl get sc`
- Describe PVC: `kubectl describe pvc <name>`
- Verify provisioner or CSI driver

### Scenario B: Pod stuck in Pending (Multi-attach)
- Describe pod to see volume attach errors
- Check access mode vs node placement

### Scenario C: Data missing after pod restart
- Ensure PVC bound and mounted
- Verify reclaim policy (Retain vs Delete)

### Scenario D: Volume mount timeout
- Check CSI controller logs
- Validate node permissions and connectivity to storage

---

## 14. Interview Questions and Answers

### Q1: PV vs PVC vs StorageClass?
**Answer:** PV is the storage resource, PVC is a user request, and StorageClass defines dynamic provisioning rules.

### Q2: Why use dynamic provisioning?
**Answer:** It avoids manual PV creation and scales storage on demand, improving agility.

### Q3: What does reclaim policy do?
**Answer:** It controls what happens to the PV after PVC deletion (Retain, Delete, Recycle).

### Q4: Explain RWO vs RWX.
**Answer:** RWO mounts read-write on one node; RWX allows read-write on many nodes, depending on backend support.

### Q5: How do StatefulSets use volumes?
**Answer:** Each replica gets its own PVC via volumeClaimTemplates, ensuring stable per-pod storage.

### Q6: Why does multi-attach happen?
**Answer:** Some volume types cannot be mounted to multiple nodes simultaneously (RWO).

### Q7: How do you expand a PVC?
**Answer:** Patch the PVC size if StorageClass allows expansion; filesystem may need resize depending on the driver.

### Q8: What is a headless service used for in StatefulSets?
**Answer:** Stable DNS per pod for direct access; not required for all volume use cases but common in stateful apps.

### Q9: What happens if a pod using a PVC is deleted?
**Answer:** The PVC remains; when the pod is recreated, it reattaches to the same volume.

### Q10: How do you protect secrets mounted as volumes?
**Answer:** Use least privilege RBAC, readOnly mounts, and avoid logging file contents.

---

## 15. Cleanup

```bash
kubectl delete ns volume-lab
```

---

## 16. Theory (Concise)

- **Ephemeral vs persistent:** Pod volumes like `emptyDir` are tied to pod lifecycle; PV-backed volumes persist.
- **Binding lifecycle:** PVCs bind to PVs; pods mount PVCs; reclaim policy defines cleanup behavior.
- **Access modes:** RWO, ROX, RWX determine node-level attach rules, not container-level file permissions.
- **Provisioning:** Static PVs are manually created; dynamic provisioning uses StorageClasses and CSI.
- **Operational edge cases:** Multi-attach errors, slow mounts, and expansion require CSI awareness and storage backend constraints.
- **Security:** Secrets/configs can be mounted as volumes but must be controlled with RBAC and readOnly where possible.

