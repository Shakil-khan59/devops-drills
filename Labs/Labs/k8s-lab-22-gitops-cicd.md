# Kubernetes Lab 22: GitOps and CI/CD

This lab introduces GitOps workflows and automated delivery.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Repo Structure for Manifests
4. Lab 2: Kustomize for Environments
5. Lab 3: GitOps Reconciliation (Concept)
6. Lab 4: Rollback via Git
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
- Optional: Argo CD or Flux installed

```bash
kubectl create ns gitops-lab
kubectl config set-context --current --namespace=gitops-lab
```

---

## 2. Core Concepts
- **GitOps:** Git is the source of truth
- **Reconciliation:** Controllers converge cluster to Git state

---

## 3. Lab 1: Repo Structure for Manifests

```
repo/
  base/
  overlays/
    dev/
    prod/
```

---

## 4. Lab 2: Kustomize for Environments

```bash
kubectl apply -k overlays/dev
kubectl apply -k overlays/prod
```

---

## 5. Lab 3: GitOps Reconciliation (Concept)

> Create an Application (Argo CD) or Kustomization (Flux) that points to the repo.

---

## 6. Lab 4: Rollback via Git

### Practice
- Revert a commit in Git
- Observe controller rollback in cluster

---

## 7. Troubleshooting Scenarios
- **Drift detected:** Manual changes in cluster differ from Git.
- **Sync errors:** Invalid manifests or missing permissions.
- **Slow rollout:** Controllers reconcile on intervals.

---

## 8. Interview Questions and Answers

### Q1: Why GitOps?
**Answer:** It provides auditability, repeatability, and automated delivery.

### Q2: How is GitOps different from CI/CD?
**Answer:** CI/CD pushes changes; GitOps pulls changes into the cluster.

### Q3: How do you handle secrets in GitOps?
**Answer:** Use sealed secrets, external secret stores, or SOPS.

---

## 9. Cleanup
```bash
kubectl delete ns gitops-lab
```

---

## 10. Theory (Concise)
- **Git is the truth**, cluster is the runtime mirror.
- **Reconciliation loops** drive drift correction.
- **Rollbacks are commits**, not imperative actions.
