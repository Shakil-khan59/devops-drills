# Kubernetes Lab 28: Service Mesh Fundamentals

This lab introduces service mesh concepts, sidecar injection, and traffic control.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Install a Mesh (Optional)
4. Lab 2: Sidecar Injection
5. Lab 3: Traffic Routing
6. Lab 4: mTLS Verification
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
- Optional: Istio or Linkerd installed

```bash
kubectl create ns mesh-lab
kubectl config set-context --current --namespace=mesh-lab
```

---

## 2. Core Concepts
- **Sidecar proxies** capture traffic
- **mTLS** encrypts service-to-service traffic
- **Traffic policies** allow retries, timeouts, canaries

---

## 3. Lab 1: Install a Mesh (Optional)

> Use `istioctl install` or `linkerd install` if tooling is available.

---

## 4. Lab 2: Sidecar Injection

```bash
kubectl label ns mesh-lab istio-injection=enabled
```

```bash
kubectl create deploy web --image=nginx:1.25
kubectl get pods
```

### Solution check
Pods have two containers (app + proxy).

---

## 5. Lab 3: Traffic Routing

> For Istio, create a VirtualService and DestinationRule to split traffic.

---

## 6. Lab 4: mTLS Verification

> Use mesh CLI or metrics to confirm mTLS is enabled.

---

## 7. Troubleshooting Scenarios
- **Sidecar missing:** Namespace not labeled or injection disabled.
- **Traffic issues:** Check VirtualService and DestinationRule.
- **mTLS errors:** Validate certificates and policy modes.

---

## 8. Interview Questions and Answers

### Q1: Why use a service mesh?
**Answer:** To standardize L7 traffic management, security, and observability.

### Q2: What is sidecar injection?
**Answer:** Automatically adds a proxy container to each pod.

### Q3: What is mTLS?
**Answer:** Mutual TLS for service-to-service authentication and encryption.

---

## 9. Cleanup
```bash
kubectl delete ns mesh-lab
```

---

## 10. Theory (Concise)
- **Meshes provide L7 controls** without app changes.
- **Sidecars implement policies** transparently.
- **mTLS enforces zero-trust** within the cluster.
