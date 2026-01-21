# Kubernetes StatefulSet Lab Tutorial (End-to-End)

This lab is a practical, end-to-end path to understanding and practicing Kubernetes StatefulSets. It includes:
- Concepts and comparisons (StatefulSet vs Deployment)
- Hands-on labs with solutions
- Storage, networking, and rollout strategies
- Debugging and operational scenarios
- Interview-style questions with sample answers

Everything is designed to be practiced locally (kind/minikube) or on any Kubernetes cluster.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Your First StatefulSet (nginx)
4. Lab 2: StatefulSet with Headless Service DNS
5. Lab 3: StatefulSet with Persistent Storage
6. Lab 4: Rolling Updates, Partitions, and Rollbacks
7. Lab 5: Scale, Delete, and Data Persistence
8. Lab 6: Pod Management Policies and Ordered Startup
9. Lab 7: Database StatefulSet (Postgres) with Readiness
10. Troubleshooting Scenarios
11. Interview Questions and Answers
12. Cleanup

---

## 1. Prerequisites and Setup

### Requirements
- Kubernetes cluster (minikube, kind, k3d, or cloud)
- kubectl
- Optional: a default StorageClass in the cluster

### Verify cluster
```bash
kubectl cluster-info
kubectl get nodes
```

### Create a namespace
```bash
kubectl create ns statefulset-lab
kubectl config set-context --current --namespace=statefulset-lab
```

---

## 2. Core Concepts

### What a StatefulSet provides
- Stable, unique network identity: pod names are predictable (e.g., web-0, web-1)
- Stable, persistent storage: PVCs per replica
- Ordered, graceful deployment and scaling

### StatefulSet vs Deployment
- Deployment: stateless, interchangeable pods
- StatefulSet: identity and storage per pod

### Key components
- Headless Service (ClusterIP: None)
- StatefulSet
- PersistentVolumeClaims (PVCs)

---

## 3. Lab 1: Your First StatefulSet (nginx)

### Goal
Create a StatefulSet with 3 replicas and verify stable pod names.

### Manifest
Save as `lab1-statefulset.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  clusterIP: None
  selector:
    app: web
  ports:
  - port: 80
    name: http
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: web
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
          name: http
```

### Apply
```bash
kubectl apply -f lab1-statefulset.yaml
kubectl get pods -l app=web
```

### Expected
Pod names are `web-0`, `web-1`, `web-2`.

---

## 4. Lab 2: StatefulSet with Headless Service DNS

### Goal
Verify stable DNS for each replica.

### Test DNS
```bash
kubectl run -it --rm dns-test --image=busybox:1.36 --restart=Never -- nslookup web-0.web
```

### Expected
Returns an IP for the web-0 pod. Repeat for web-1, web-2.

---

## 5. Lab 3: StatefulSet with Persistent Storage

### Goal
Attach per-pod persistent storage.

### Manifest
Save as `lab3-storage.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: data-web
spec:
  clusterIP: None
  selector:
    app: data-web
  ports:
  - port: 80
    name: http
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: data-web
spec:
  serviceName: data-web
  replicas: 2
  selector:
    matchLabels:
      app: data-web
  template:
    metadata:
      labels:
        app: data-web
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

### Apply
```bash
kubectl apply -f lab3-storage.yaml
kubectl get pvc
```

### Verify persistence
```bash
kubectl exec -it data-web-0 -- sh -c 'echo "hello from web-0" > /usr/share/nginx/html/index.html'
kubectl exec -it data-web-0 -- cat /usr/share/nginx/html/index.html
```

---

## 6. Lab 4: Rolling Updates, Partitions, and Rollbacks

### Goal
Control updates and roll back safely.

### Update image
```bash
kubectl set image statefulset/data-web nginx=nginx:1.26
kubectl rollout status statefulset/data-web
```

### Partitioned rollout
```bash
kubectl patch statefulset data-web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":1}}}}'
```
This updates pods with ordinal >= 1 (e.g., data-web-1), leaving data-web-0 unchanged.

### Rollback
```bash
kubectl rollout undo statefulset/data-web
```

---

## 7. Lab 5: Scale, Delete, and Data Persistence

### Goal
Show data persists when pods are recreated.

### Scale down
```bash
kubectl scale statefulset data-web --replicas=0
kubectl get pods -l app=data-web
```

### Scale up
```bash
kubectl scale statefulset data-web --replicas=2
kubectl get pods -l app=data-web
```

### Verify data still exists
```bash
kubectl exec -it data-web-0 -- cat /usr/share/nginx/html/index.html
```

---

## 8. Lab 6: Pod Management Policies and Ordered Startup

### Goal
Understand ordered vs parallel pod creation.

### Patch to parallel
```bash
kubectl patch statefulset data-web -p '{"spec":{"podManagementPolicy":"Parallel"}}'
```

### Scale and observe
```bash
kubectl scale statefulset data-web --replicas=3
kubectl get pods -l app=data-web -w
```

---

## 9. Lab 7: Database StatefulSet (Postgres) with Readiness

### Goal
Practice a real stateful workload with readiness and persistence.

### Manifest
Save as `lab7-postgres.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: pg
spec:
  clusterIP: None
  selector:
    app: pg
  ports:
  - port: 5432
    name: pg
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: pg
spec:
  serviceName: pg
  replicas: 2
  selector:
    matchLabels:
      app: pg
  template:
    metadata:
      labels:
        app: pg
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_PASSWORD
          value: example
        - name: POSTGRES_DB
          value: labdb
        ports:
        - containerPort: 5432
          name: pg
        readinessProbe:
          exec:
            command: ["/bin/sh", "-c", "pg_isready -U postgres"]
          initialDelaySeconds: 5
          periodSeconds: 10
        volumeMounts:
        - name: pgdata
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: pgdata
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 2Gi
```

### Apply
```bash
kubectl apply -f lab7-postgres.yaml
kubectl get pods -l app=pg
```

### Test connectivity
```bash
kubectl run -it --rm pg-client --image=postgres:15 --restart=Never -- \
  psql -h pg-0.pg -U postgres -d labdb
```

---

## 10. Troubleshooting Scenarios

### Scenario A: Pods stuck in Pending
- Check PVCs: `kubectl get pvc`
- Check StorageClass: `kubectl get sc`
- Describe PVC: `kubectl describe pvc <name>`

### Scenario B: Pod cannot resolve peer DNS
- Ensure Service is headless (ClusterIP: None)
- Validate selector matches pod labels

### Scenario C: Data lost after restart
- Check if PVCs were deleted
- Verify mountPath and volumeClaimTemplates

### Scenario D: Rolling update stuck
- Check readiness probes
- Describe pod and StatefulSet events

---

## 11. Interview Questions and Answers

### Q1: When should you use a StatefulSet instead of a Deployment?
**Answer:** When pods need stable identity, stable storage, or ordered rollouts, such as databases, queues, or stateful apps.

### Q2: What does a headless Service do for a StatefulSet?
**Answer:** It provides stable DNS entries for each pod (podname.servicename), enabling direct pod-to-pod access.

### Q3: How does storage work in StatefulSets?
**Answer:** Each replica gets its own PVC from volumeClaimTemplates, ensuring data persistence across restarts.

### Q4: What happens if you delete a StatefulSet?
**Answer:** Pods are removed, but PVCs are retained by default unless deleted manually.

### Q5: How do you perform a canary rollout for StatefulSet?
**Answer:** Use updateStrategy with partition to update only higher ordinals first.

### Q6: What are pod management policies?
**Answer:** OrderedReady creates pods one by one; Parallel creates them simultaneously.

### Q7: How do you do a safe scale down?
**Answer:** StatefulSet removes highest ordinal first, ensuring predictable shutdown order.

### Q8: Why might a StatefulSet be stuck during rollout?
**Answer:** Readiness probes not passing or persistent volume issues.

### Q9: Can two StatefulSet replicas share the same PVC?
**Answer:** Not by default. Each replica gets its own PVC. Sharing usually requires ReadWriteMany volumes and custom configuration.

### Q10: What happens to DNS if a pod is deleted and recreated?
**Answer:** The pod name and DNS remain consistent (e.g., web-0.web), though the IP may change.

---

## 12. Cleanup

```bash
kubectl delete ns statefulset-lab
```

---

## Notes and Extensions

Suggested follow-ups to deepen learning:
- Add anti-affinity rules to spread StatefulSet pods across nodes.
- Use init containers to bootstrap per-pod configuration.
- Explore volume expansion and StorageClass options.
- Practice backup and restore for persistent volumes.

---

## 13. Theory (Concise)

- **Stateful identity:** Each replica has a stable name and hostname (e.g., `web-0`), which is critical for leader/follower patterns and direct addressing.
- **Stable storage:** `volumeClaimTemplates` create one PVC per replica; PVCs persist even if pods are deleted.
- **Headless service:** Required for stable DNS records `podname.servicename`, enabling predictable peer discovery.
- **Ordered lifecycle:** Pods are created, updated, and terminated in ordinal order unless `podManagementPolicy: Parallel` is set.
- **Update control:** Rolling updates can be partitioned to control which ordinals update first; rollbacks are supported via `kubectl rollout undo`.
- **Scaling behavior:** Scaling down removes higher ordinals first; scaling up reuses existing PVCs, preserving data.
