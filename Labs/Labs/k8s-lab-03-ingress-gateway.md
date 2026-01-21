# Kubernetes Lab 03: Ingress and Gateway

This lab covers HTTP routing, Ingress rules, TLS, and gateway patterns.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Install or Verify Ingress Controller
4. Lab 2: Host-Based Routing
5. Lab 3: Path-Based Routing
6. Lab 4: TLS Termination
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
- Ingress controller (NGINX, Traefik, or cloud)

```bash
kubectl create ns ingress-lab
kubectl config set-context --current --namespace=ingress-lab
```

---

## 2. Core Concepts
- **Ingress:** L7 routing to Services
- **IngressClass:** Binds rules to a controller
- **Gateway API:** Newer, more expressive alternative

---

## 3. Lab 1: Install or Verify Ingress Controller

> If already installed, skip.

```bash
kubectl get pods -A | rg -n "ingress|nginx|traefik"
```

### Solution check
An ingress controller pod is running.

---

## 4. Lab 2: Host-Based Routing

### Create two services
```bash
kubectl create deploy app-a --image=nginx:1.25
kubectl expose deploy app-a --port=80
kubectl create deploy app-b --image=nginx:1.25
kubectl expose deploy app-b --port=80
```

### Ingress (`lab2-host.yaml`)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-ingress
spec:
  rules:
  - host: a.lab.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-a
            port:
              number: 80
  - host: b.lab.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-b
            port:
              number: 80
```

### Apply and test
```bash
kubectl apply -f lab2-host.yaml
```

> Use `/etc/hosts` or curl `-H "Host: a.lab.local"` with the ingress IP.

---

## 5. Lab 3: Path-Based Routing

### Ingress (`lab3-path.yaml`)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-ingress
spec:
  rules:
  - host: app.lab.local
    http:
      paths:
      - path: /a
        pathType: Prefix
        backend:
          service:
            name: app-a
            port:
              number: 80
      - path: /b
        pathType: Prefix
        backend:
          service:
            name: app-b
            port:
              number: 80
```

---

## 6. Lab 4: TLS Termination

### Create self-signed cert
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=app.lab.local"

kubectl create secret tls app-tls --cert=tls.crt --key=tls.key
```

### TLS Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
    - app.lab.local
    secretName: app-tls
  rules:
  - host: app.lab.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-a
            port:
              number: 80
```

---

## 7. Troubleshooting Scenarios
- **No controller:** Ingress rules exist but no traffic flows.
- **404 from controller:** Rule mismatch or wrong pathType.
- **TLS errors:** Incorrect secret type or cert mismatch.

---

## 8. Interview Questions and Answers

### Q1: Ingress vs Service?
**Answer:** Service provides L4 access; Ingress provides L7 routing and TLS termination.

### Q2: What is IngressClass?
**Answer:** It selects the controller that will reconcile the Ingress.

### Q3: Why use Gateway API?
**Answer:** It adds stronger role separation and better extensibility.

---

## 9. Cleanup
```bash
kubectl delete ns ingress-lab
```

---

## 10. Theory (Concise)
- **Ingress routes HTTP/S traffic** to Services using rules.
- **Controllers implement Ingress**, not the API alone.
- **TLS termination** is handled at the Ingress layer.
- **Gateway API** is the evolution path for L7 routing.
