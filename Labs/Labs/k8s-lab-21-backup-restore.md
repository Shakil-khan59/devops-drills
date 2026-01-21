# Kubernetes Lab 21: Backup and Restore (Velero and Manual)

This lab covers backup strategies for manifests and persistent data, with optional Velero workflows.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Export Manifests (Manual Backup)
4. Lab 2: App-Level Data Backup (Example)
5. Lab 3: Velero Backup (If Installed)
6. Lab 4: Restore and Validation
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
- Optional: Velero installed and configured

```bash
kubectl create ns backup-lab
kubectl config set-context --current --namespace=backup-lab
```

---

## 2. Core Concepts
- **Manifests backup:** Cluster state definitions
- **Data backup:** PV contents or app-level snapshots
- **Recovery time:** Measured via RTO/RPO objectives

---

## 3. Lab 1: Export Manifests (Manual Backup)

```bash
kubectl get all -o yaml > backup-lab-manifests.yaml
kubectl get pvc -o yaml >> backup-lab-manifests.yaml
```

### Solution check
A single YAML file contains core resources for the namespace.

---

## 4. Lab 2: App-Level Data Backup (Example)

```bash
kubectl run -it --rm pg-client --image=postgres:15 --restart=Never -- \
  sh -c "pg_dump -h <db-host> -U postgres -d labdb > /tmp/backup.sql"
```

### Solution check
A SQL dump file is created (app-level backup).

---

## 5. Lab 3: Velero Backup (If Installed)

```bash
velero backup create backup-lab --include-namespaces backup-lab
velero backup get
```

---

## 6. Lab 4: Restore and Validation

```bash
velero restore create --from-backup backup-lab
kubectl get all
```

### Solution check
Resources return and workloads recover.

---

## 7. Troubleshooting Scenarios
- **Missing PV data:** Backup tool not configured for volume snapshots.
- **Restore fails:** Verify CRDs and Velero server logs.
- **App inconsistency:** Use app-level quiescing or transactional backups.

---

## 8. Interview Questions and Answers

### Q1: Why are manifest backups not enough?
**Answer:** Manifests restore configuration but not persistent data.

### Q2: What is RPO?
**Answer:** The maximum acceptable data loss window.

### Q3: How do you back up stateful apps safely?
**Answer:** Use app-level tools plus storage snapshots.

---

## 9. Cleanup
```bash
kubectl delete ns backup-lab
rm -f backup-lab-manifests.yaml
```

---

## 10. Theory (Concise)
- **Backups require both state and data** for full recovery.
- **Snapshots are fast**, app-level dumps are consistent.
- **RTO/RPO drive strategy** and tooling choices.
