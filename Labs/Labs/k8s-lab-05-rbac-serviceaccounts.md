# Kubernetes Lab 05: RBAC and ServiceAccounts

This lab focuses on identity, permissions, and least-privilege access.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: ServiceAccount Basics
4. Lab 2: Role and RoleBinding
5. Lab 3: Test Permissions
6. Lab 4: Pod Using a ServiceAccount
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns rbac-lab
kubectl config set-context --current --namespace=rbac-lab
```

---

## 2. Core Concepts
- **ServiceAccount:** Identity for pods
- **Role/ClusterRole:** Permissions
- **RoleBinding/ClusterRoleBinding:** Attach permissions to subjects

---

## 3. Lab 1: ServiceAccount Basics
```bash
kubectl create serviceaccount app-sa
kubectl get sa
```

---

## 4. Lab 2: Role and RoleBinding

### Role (`lab2-role.yaml`)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

### RoleBinding (`lab2-binding.yaml`)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
subjects:
- kind: ServiceAccount
  name: app-sa
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Apply
```bash
kubectl apply -f lab2-role.yaml
kubectl apply -f lab2-binding.yaml
```

---

## 5. Lab 3: Test Permissions
```bash
kubectl auth can-i list pods --as=system:serviceaccount:rbac-lab:app-sa
kubectl auth can-i delete pods --as=system:serviceaccount:rbac-lab:app-sa
```

### Solution check
List is allowed; delete is denied.

---

## 6. Lab 4: Pod Using a ServiceAccount

```bash
kubectl apply -f - <<'YAML'
apiVersion: v1
kind: Pod
metadata:
  name: sa-demo
spec:
  serviceAccountName: app-sa
  containers:
  - name: app
    image: bitnami/kubectl:1.28
    command: ["/bin/sh", "-c", "kubectl get pods; sleep 3600"]
YAML
```

### Solution check
Pod can list pods in the namespace.

---

## 7. Troubleshooting Scenarios
- **Forbidden errors:** Check RoleBinding namespace and subject.
- **Cluster-wide access:** Use ClusterRole and ClusterRoleBinding.
- **Token issues:** Ensure automountServiceAccountToken is true if needed.

---

## 8. Interview Questions and Answers

### Q1: Role vs ClusterRole?
**Answer:** Role is namespace-scoped; ClusterRole can be used cluster-wide.

### Q2: Why use ServiceAccounts?
**Answer:** They provide identity and scoped permissions for pods.

### Q3: How do you test RBAC rules quickly?
**Answer:** `kubectl auth can-i` with impersonation flags.

---

## 9. Cleanup
```bash
kubectl delete ns rbac-lab
```

---

## 10. Theory (Concise)
- **RBAC enforces least privilege** for users and workloads.
- **Bindings attach permissions** to subjects.
- **ServiceAccounts are pod identities** with scoped access.
