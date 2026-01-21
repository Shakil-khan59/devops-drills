# Kubernetes Lab 01: Deployments and ReplicaSets

This lab builds practical mastery of Deployments, ReplicaSets, rollouts, and safe release patterns.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Create and Scale a Deployment
4. Lab 2: Rolling Update and Rollback
5. Lab 3: Blue/Green with Two Deployments
6. Lab 4: Canary with Label Switching
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns deploy-lab
kubectl config set-context --current --namespace=deploy-lab
```

---

## 2. Core Concepts
- **Deployment:** Declarative updates for Pods and ReplicaSets
- **ReplicaSet:** Ensures the specified number of Pods
- **Rollout history:** Built-in revision tracking for safe rollbacks

---

## 3. Lab 1: Create and Scale a Deployment

### Goal
Create a Deployment and scale it to verify ReplicaSet behavior.

### Manifest (`lab1-deploy.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
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
```

### Apply
```bash
kubectl apply -f lab1-deploy.yaml
kubectl get deploy,rs,pods -l app=web
```

### Scale
```bash
kubectl scale deploy/web --replicas=4
kubectl get rs,pods -l app=web
```

### Solution check
Pods scale to 4; ReplicaSet remains 1.

---

## 4. Lab 2: Rolling Update and Rollback

### Update image
```bash
kubectl set image deploy/web nginx=nginx:1.26
kubectl rollout status deploy/web
```

### View history
```bash
kubectl rollout history deploy/web
```

### Rollback
```bash
kubectl rollout undo deploy/web
```

### Solution check
Pods return to the previous nginx version.

---

## 5. Lab 3: Blue/Green with Two Deployments

### Goal
Run two Deployments and switch traffic with a Service selector.

### Create blue and green
```bash
kubectl create deploy web-blue --image=nginx:1.25 --replicas=2
kubectl label deploy web-blue app=web track=blue
kubectl create deploy web-green --image=nginx:1.26 --replicas=2
kubectl label deploy web-green app=web track=green
```

### Service (`lab3-svc.yaml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
    track: blue
  ports:
  - port: 80
    targetPort: 80
```

### Switch traffic to green
```bash
kubectl patch svc web -p '{"spec":{"selector":{"app":"web","track":"green"}}}'
```

### Solution check
Service endpoints now point to green pods.

---

## 6. Lab 4: Canary with Label Switching

### Goal
Create a canary Deployment and gradually shift traffic.

### Canary
```bash
kubectl create deploy web-canary --image=nginx:1.26 --replicas=1
kubectl label deploy web-canary app=web track=canary
```

### Patch service selector temporarily
```bash
kubectl patch svc web -p '{"spec":{"selector":{"app":"web"}}}'
```

### Solution check
Service now targets all pods (blue/green/canary). Use readiness and metrics to validate.

---

## 7. Troubleshooting Scenarios
- **Pods not updated:** Check `kubectl rollout status` and Deployment events.
- **CrashLoopBackOff:** Inspect `kubectl logs` and probe settings.
- **Rollback not working:** Ensure rollout history exists and revision limits are sufficient.

---

## 8. Interview Questions and Answers

### Q1: Difference between Deployment and ReplicaSet?
**Answer:** Deployment manages ReplicaSets and provides rollout/rollback; ReplicaSet only maintains replica count.

### Q2: How does a rolling update work?
**Answer:** It incrementally replaces old pods with new ones using `maxUnavailable` and `maxSurge`.

### Q3: How do you do blue/green without a service mesh?
**Answer:** Run two Deployments with labels and switch the Service selector.

### Q4: How do you undo a bad release?
**Answer:** Use `kubectl rollout undo deploy/<name>` to revert to the prior revision.

---

## 9. Cleanup
```bash
kubectl delete ns deploy-lab
```

---

## 10. Theory (Concise)
- **Deployment = control plane for ReplicaSets**, enabling safe rollouts and rollbacks.
- **ReplicaSet enforces replica count**, but has no rollout logic.
- **Rollouts are progressive**, guided by surge/unavailable limits.
- **Traffic shifting** can be done with Services or a mesh.
- **Revision history** protects against bad releases.
