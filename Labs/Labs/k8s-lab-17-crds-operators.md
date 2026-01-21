# Kubernetes Lab 17: CRDs and Operators

This lab introduces custom resources, schemas, and the operator pattern.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Create a CRD
4. Lab 2: Create a Custom Resource
5. Lab 3: Validate Schema
6. Lab 4: Operator Workflow (Conceptual)
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns crd-lab
kubectl config set-context --current --namespace=crd-lab
```

---

## 2. Core Concepts
- **CRD:** Extends the Kubernetes API
- **CR:** An instance of a custom resource
- **Operator:** Controller that reconciles CRs

---

## 3. Lab 1: Create a CRD

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: backups.example.com
spec:
  group: example.com
  names:
    kind: Backup
    plural: backups
    singular: backup
    shortNames: ["bkp"]
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              target:
                type: string
              schedule:
                type: string
```

### Apply
```bash
kubectl apply -f - <<'YAML'
<PASTE THE YAML ABOVE>
YAML
```

---

## 4. Lab 2: Create a Custom Resource

```yaml
apiVersion: example.com/v1
kind: Backup
metadata:
  name: db-backup
spec:
  target: postgres
  schedule: "0 2 * * *"
```

### Apply
```bash
kubectl apply -f - <<'YAML'
<PASTE THE YAML ABOVE>
YAML
kubectl get backups
```

---

## 5. Lab 3: Validate Schema

Try a bad spec:
```yaml
apiVersion: example.com/v1
kind: Backup
metadata:
  name: bad-backup
spec:
  target: 123
```

### Solution check
Validation fails because target is not a string.

---

## 6. Lab 4: Operator Workflow (Conceptual)

> Operators watch CRs and reconcile desired state. Use Kubebuilder or Operator SDK to generate controller scaffolding.

---

## 7. Troubleshooting Scenarios
- **CRD not served:** Check `spec.versions`.
- **Validation failures:** Inspect schema under `openAPIV3Schema`.
- **No controller action:** Operator not installed.

---

## 8. Interview Questions and Answers

### Q1: What is a CRD?
**Answer:** A way to add new API types to Kubernetes.

### Q2: Why use an operator?
**Answer:** It automates lifecycle management of complex apps.

### Q3: How are CRDs versioned?
**Answer:** Using multiple versions in `spec.versions` with one marked `storage`.

---

## 9. Cleanup
```bash
kubectl delete ns crd-lab
kubectl delete crd backups.example.com
```

---

## 10. Theory (Concise)
- **CRDs extend the API**, CRs hold desired state.
- **Operators reconcile CRs** into concrete resources.
- **Schemas enforce validation** and API contracts.
