# Kubernetes Lab 16: Kustomize

This lab teaches environment overlays, patching, and declarative customization.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Base Configuration
4. Lab 2: Overlay for Staging
5. Lab 3: Image and Name Changes
6. Lab 4: ConfigMap Generator
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns kustomize-lab
kubectl config set-context --current --namespace=kustomize-lab
```

---

## 2. Core Concepts
- **Base:** Shared resources
- **Overlay:** Environment-specific patches
- **Kustomization:** Declares how to build output

---

## 3. Lab 1: Base Configuration

Create `kustomize/base/deploy.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: nginx:1.25
```

Create `kustomize/base/kustomization.yaml`:
```yaml
resources:
- deploy.yaml
```

---

## 4. Lab 2: Overlay for Staging

Create `kustomize/overlays/staging/kustomization.yaml`:
```yaml
resources:
- ../../base
patchesStrategicMerge:
- replica-patch.yaml
```

Create `replica-patch.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
```

### Apply
```bash
kubectl apply -k kustomize/overlays/staging
```

---

## 5. Lab 3: Image and Name Changes

Add to overlay `kustomization.yaml`:
```yaml
images:
- name: nginx
  newTag: "1.26"
nameSuffix: "-stg"
```

---

## 6. Lab 4: ConfigMap Generator

Add to base `kustomization.yaml`:
```yaml
configMapGenerator:
- name: app-config
  literals:
  - MODE=staging
```

### Solution check
ConfigMap name is hashed for immutability.

---

## 7. Troubleshooting Scenarios
- **Patch not applied:** Check kind/name match.
- **Hash changes:** ConfigMapGenerator changes name on data update.
- **Overlays drift:** Ensure base is the single source.

---

## 8. Interview Questions and Answers

### Q1: Kustomize vs Helm?
**Answer:** Kustomize patches plain YAML; Helm uses templates and releases.

### Q2: Why does ConfigMapGenerator add a hash?
**Answer:** To force pod restart when config changes.

### Q3: How do overlays work?
**Answer:** They layer patches and transformations on top of base resources.

---

## 9. Cleanup
```bash
kubectl delete ns kustomize-lab
rm -rf kustomize
```

---

## 10. Theory (Concise)
- **Kustomize is template-free**, patch-based customization.
- **Overlays capture environment differences** cleanly.
- **Generators support immutable config updates**.
