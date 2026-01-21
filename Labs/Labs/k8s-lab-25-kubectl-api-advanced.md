# Kubernetes Lab 25: kubectl and API Fundamentals (Advanced)

This lab sharpens API discovery, output control, and server-side apply.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: API Discovery and Explain
4. Lab 2: JSONPath and Custom Columns
5. Lab 3: Server-Side Apply and Diff
6. Lab 4: Field Selectors and Labels
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns api-lab
kubectl config set-context --current --namespace=api-lab
```

---

## 2. Core Concepts
- **Discovery:** `api-resources`, `api-versions`
- **Explain:** Schema and field docs
- **Apply:** Server-side vs client-side

---

## 3. Lab 1: API Discovery and Explain

```bash
kubectl api-resources | rg -n "deploy|stateful|job"
kubectl explain deploy.spec.strategy
```

---

## 4. Lab 2: JSONPath and Custom Columns

```bash
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o custom-columns=NAME:.metadata.name,PHASE:.status.phase
```

---

## 5. Lab 3: Server-Side Apply and Diff

```bash
kubectl apply --server-side -f <file>
kubectl diff -f <file>
```

---

## 6. Lab 4: Field Selectors and Labels

```bash
kubectl get pods --field-selector=status.phase=Running
kubectl get pods -l app=web
```

---

## 7. Troubleshooting Scenarios
- **Apply conflicts:** Check managedFields and field ownership.
- **Unexpected output:** Validate JSONPath syntax.
- **No resources:** Ensure namespace context.

---

## 8. Interview Questions and Answers

### Q1: What is server-side apply?
**Answer:** The API server merges fields and tracks field ownership.

### Q2: How do you inspect resource schema?
**Answer:** Use `kubectl explain`.

### Q3: Difference between label and field selectors?
**Answer:** Labels are user-defined; fields are resource status/spec fields.

---

## 9. Cleanup
```bash
kubectl delete ns api-lab
```

---

## 10. Theory (Concise)
- **kubectl is an API client**, not the source of truth.
- **Server-side apply enables collaboration** on manifests.
- **Selectors improve operational queries** and automation.
