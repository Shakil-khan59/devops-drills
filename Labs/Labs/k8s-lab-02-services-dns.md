# Kubernetes Lab 02: Services and DNS

This lab builds mastery of Service types, DNS, and stable access patterns.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: ClusterIP Service
4. Lab 2: Headless Service and DNS
5. Lab 3: NodePort Service
6. Lab 4: LoadBalancer Service (If Supported)
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns svc-lab
kubectl config set-context --current --namespace=svc-lab
```

---

## 2. Core Concepts
- **Service:** Stable virtual IP and DNS for pods
- **Selectors:** Bind Service to pod labels
- **Types:** ClusterIP, NodePort, LoadBalancer, ExternalName

---

## 3. Lab 1: ClusterIP Service

### Create a Deployment
```bash
kubectl create deploy web --image=nginx:1.25 --replicas=2
kubectl label deploy web app=web
```

### Expose as ClusterIP
```bash
kubectl expose deploy web --port=80 --target-port=80
kubectl get svc web
```

### Verify
```bash
kubectl run -it --rm curl --image=curlimages/curl:8.5.0 --restart=Never -- \
  curl -s web
```

### Solution check
Returns the nginx default HTML.

---

## 4. Lab 2: Headless Service and DNS

### Headless Service
```bash
kubectl apply -f - <<'YAML'
apiVersion: v1
kind: Service
metadata:
  name: web-headless
spec:
  clusterIP: None
  selector:
    app: web
  ports:
  - port: 80
YAML
```

### DNS check
```bash
kubectl run -it --rm dns --image=busybox:1.36 --restart=Never -- \
  nslookup web-headless
```

### Solution check
DNS returns multiple A records, one per pod.

---

## 5. Lab 3: NodePort Service

```bash
kubectl expose deploy web --type=NodePort --name=web-nodeport --port=80
kubectl get svc web-nodeport
```

### Solution check
Use any node IP plus NodePort to access the service (if nodes are reachable).

---

## 6. Lab 4: LoadBalancer Service (If Supported)

```bash
kubectl expose deploy web --type=LoadBalancer --name=web-lb --port=80
kubectl get svc web-lb -w
```

### Solution check
An external IP is assigned if the environment supports it.

---

## 7. Troubleshooting Scenarios
- **No endpoints:** Verify pod labels match service selector.
- **DNS failure:** Check kube-dns/CoreDNS health.
- **NodePort unreachable:** Ensure node firewall allows the port.

---

## 8. Interview Questions and Answers

### Q1: What is the purpose of ClusterIP?
**Answer:** It provides stable virtual IP for internal cluster traffic.

### Q2: What does a headless service do?
**Answer:** It disables VIP load-balancing and returns pod IPs directly via DNS.

### Q3: When use NodePort?
**Answer:** For simple external access without a cloud load balancer.

### Q4: How does kube-proxy implement Services?
**Answer:** It programs iptables or IPVS rules to route traffic.

---

## 9. Cleanup
```bash
kubectl delete ns svc-lab
```

---

## 10. Theory (Concise)
- **Services provide stable addressing** across pod restarts.
- **Headless services enable direct pod discovery** for stateful systems.
- **Service type controls exposure** and traffic flow.
- **DNS integrates with Kubernetes** for predictable service names.
