# Kubernetes Lab 04: ConfigMaps and Secrets

This lab teaches configuration injection, secret handling, and safe rollout patterns.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: ConfigMap as Env Vars
4. Lab 2: ConfigMap as Files
5. Lab 3: Secrets as Files
6. Lab 4: Rolling Restart on Config Change
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns config-lab
kubectl config set-context --current --namespace=config-lab
```

---

## 2. Core Concepts
- **ConfigMap:** Non-sensitive config
- **Secret:** Sensitive data, base64-encoded
- **Projection:** Mount config and secrets as files

---

## 3. Lab 1: ConfigMap as Env Vars

```bash
kubectl create configmap app-config --from-literal=APP_MODE=prod
kubectl apply -f - <<'YAML'
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "echo $APP_MODE; sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config
YAML
```

### Solution check
`kubectl logs env-demo` prints `prod`.

---

## 4. Lab 2: ConfigMap as Files

```bash
kubectl create configmap file-config --from-literal=app.conf=mode=prod
kubectl apply -f - <<'YAML'
apiVersion: v1
kind: Pod
metadata:
  name: file-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "cat /etc/config/app.conf; sleep 3600"]
    volumeMounts:
    - name: cfg
      mountPath: /etc/config
  volumes:
  - name: cfg
    configMap:
      name: file-config
YAML
```

---

## 5. Lab 3: Secrets as Files

```bash
kubectl create secret generic app-secret --from-literal=API_KEY=xyz123
kubectl apply -f - <<'YAML'
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "cat /etc/secret/API_KEY; sleep 3600"]
    volumeMounts:
    - name: secret
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret
    secret:
      secretName: app-secret
YAML
```

---

## 6. Lab 4: Rolling Restart on Config Change

### Update ConfigMap
```bash
kubectl create configmap app-config --from-literal=APP_MODE=staging -o yaml --dry-run=client | kubectl apply -f -
```

### Trigger rollout
```bash
kubectl rollout restart deploy/<your-deploy>
```

---

## 7. Troubleshooting Scenarios
- **Config not updated:** Pods must be restarted to pick new env vars.
- **Permission denied:** Use `readOnly: true` for secrets.
- **Missing keys:** Validate ConfigMap/Secret names and keys.

---

## 8. Interview Questions and Answers

### Q1: Why not store secrets in ConfigMaps?
**Answer:** Secrets are designed for sensitive data and can be encrypted at rest.

### Q2: How do updates propagate?
**Answer:** Mounted files update automatically; env vars require pod restart.

### Q3: How do you avoid leaking secrets?
**Answer:** Use least privilege RBAC and avoid logging secret files.

---

## 9. Cleanup
```bash
kubectl delete ns config-lab
```

---

## 10. Theory (Concise)
- **ConfigMaps handle non-sensitive config**, Secrets handle sensitive data.
- **Mounting as files** enables live updates in many cases.
- **Env vars require restart** for updates.
- **Security depends on RBAC and encryption** at rest.
