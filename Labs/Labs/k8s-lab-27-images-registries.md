# Kubernetes Lab 27: Images, Registries, and Pull Policies

This lab covers image tags, digests, pull policies, and private registries.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Image Tags and Pull Policy
4. Lab 2: Pin by Digest
5. Lab 3: Private Registry Secret
6. Lab 4: Image Caching and Rollouts
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns images-lab
kubectl config set-context --current --namespace=images-lab
```

---

## 2. Core Concepts
- **Tags are mutable**, digests are immutable
- **imagePullPolicy:** Always, IfNotPresent, Never

---

## 3. Lab 1: Image Tags and Pull Policy

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tag-demo
spec:
  containers:
  - name: app
    image: nginx:1.25
    imagePullPolicy: IfNotPresent
```

---

## 4. Lab 2: Pin by Digest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: digest-demo
spec:
  containers:
  - name: app
    image: nginx@sha256:<DIGEST>
```

---

## 5. Lab 3: Private Registry Secret

```bash
kubectl create secret docker-registry regcred \
  --docker-server=<REGISTRY> \
  --docker-username=<USER> \
  --docker-password=<PASS>
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-demo
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: app
    image: <REGISTRY>/<IMAGE>:<TAG>
```

---

## 6. Lab 4: Image Caching and Rollouts

### Practice
- Deploy with `IfNotPresent`
- Update tag and observe pull behavior

---

## 7. Troubleshooting Scenarios
- **ImagePullBackOff:** Wrong image name or registry auth.
- **Latest tag surprises:** Use explicit versions or digests.
- **Slow pulls:** Use node-local caching or registry mirrors.

---

## 8. Interview Questions and Answers

### Q1: Why pin images by digest?
**Answer:** It guarantees immutability and repeatable deploys.

### Q2: When use imagePullPolicy Always?
**Answer:** For mutable tags or frequent updates.

### Q3: How do you configure private registry access?
**Answer:** Use `imagePullSecrets` with a registry credential secret.

---

## 9. Cleanup
```bash
kubectl delete ns images-lab
```

---

## 10. Theory (Concise)
- **Tags are mutable**, digests are authoritative.
- **Pull policy affects rollout speed** and consistency.
- **Private registries require auth** via imagePullSecrets.
