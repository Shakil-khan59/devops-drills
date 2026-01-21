# Kubernetes Lab 20: Debugging and Ephemeral Containers

This lab teaches practical debugging workflows for failing workloads.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Inspect a Failing Pod
4. Lab 2: Exec and Port-Forward
5. Lab 3: Ephemeral Containers (If Supported)
6. Lab 4: kubectl debug with a Copy
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns debug-lab
kubectl config set-context --current --namespace=debug-lab
```

---

## 2. Core Concepts
- **Describe and events** show why things fail
- **Exec and port-forward** give fast access
- **Ephemeral containers** aid live debugging

---

## 3. Lab 1: Inspect a Failing Pod

```bash
kubectl apply -f - <<'YAML'
apiVersion: v1
kind: Pod
metadata:
  name: bad-image
spec:
  containers:
  - name: app
    image: nginx:does-not-exist
YAML

kubectl describe pod bad-image
```

---

## 4. Lab 2: Exec and Port-Forward

```bash
kubectl create deploy web --image=nginx:1.25
kubectl port-forward deploy/web 8080:80
```

---

## 5. Lab 3: Ephemeral Containers (If Supported)

```bash
kubectl debug -it deploy/web --image=busybox:1.36 --target=nginx -- /bin/sh
```

---

## 6. Lab 4: kubectl debug with a Copy

```bash
kubectl debug pod/bad-image -it --image=busybox:1.36 --copy-to=bad-image-debug -- /bin/sh
```

---

## 7. Troubleshooting Scenarios
- **ImagePullBackOff:** Check image name and registry access.
- **CrashLoopBackOff:** Inspect logs and command/args.
- **Pending pods:** Check scheduling events and resource requests.

---

## 8. Interview Questions and Answers

### Q1: What is the fastest way to see why a pod failed?
**Answer:** `kubectl describe pod` and events, then logs.

### Q2: When use ephemeral containers?
**Answer:** When the original container lacks debugging tools.

### Q3: What is the difference between exec and debug?
**Answer:** Exec enters the running container; debug adds a new container or copies the pod.

---

## 9. Cleanup
```bash
kubectl delete ns debug-lab
```

---

## 10. Theory (Concise)
- **Describe + events** are the first line of diagnosis.
- **Ephemeral containers** improve live debugging without rebuilding images.
- **Copy-to debug** isolates investigation safely.
