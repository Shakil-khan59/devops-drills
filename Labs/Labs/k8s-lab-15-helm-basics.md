# Kubernetes Lab 15: Helm Basics

This lab introduces Helm charts, templating, and release management.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Create a Chart
4. Lab 2: Install a Release
5. Lab 3: Upgrade and Rollback
6. Lab 4: Values and Overrides
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
- Helm installed (`helm version`)

```bash
kubectl create ns helm-lab
kubectl config set-context --current --namespace=helm-lab
```

---

## 2. Core Concepts
- **Chart:** A package of Kubernetes manifests
- **Release:** A deployed instance of a chart
- **Values:** Configuration inputs for templates

---

## 3. Lab 1: Create a Chart

```bash
helm create demo-chart
ls demo-chart
```

---

## 4. Lab 2: Install a Release

```bash
helm install demo demo-chart
kubectl get all
```

---

## 5. Lab 3: Upgrade and Rollback

```bash
helm upgrade demo demo-chart --set replicaCount=2
helm history demo
helm rollback demo 1
```

---

## 6. Lab 4: Values and Overrides

```bash
cat <<'YAML' > values-override.yaml
replicaCount: 3
image:
  tag: "1.25"
YAML

helm upgrade demo demo-chart -f values-override.yaml
```

---

## 7. Troubleshooting Scenarios
- **Failed release:** Use `helm status` and `helm history`.
- **Template errors:** Run `helm template` locally.
- **Resource conflicts:** Check name overrides and existing objects.

---

## 8. Interview Questions and Answers

### Q1: Helm vs kubectl apply?
**Answer:** Helm adds templating, versioning, and release management.

### Q2: What is a Helm release?
**Answer:** A deployed instance of a chart with its own state.

### Q3: How do you debug templates?
**Answer:** Use `helm template` or `helm lint`.

---

## 9. Cleanup
```bash
kubectl delete ns helm-lab
rm -rf demo-chart values-override.yaml
```

---

## 10. Theory (Concise)
- **Helm packages apps** for consistent deployments.
- **Values customize environments** without editing templates.
- **Release history enables rollbacks** for safe changes.
