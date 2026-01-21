# Kubernetes Lab 07: Network Policies

This lab teaches network segmentation and traffic control at L3/L4.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Default Deny Ingress
4. Lab 2: Allow Same-Namespace Traffic
5. Lab 3: Allow From Specific Namespace
6. Lab 4: Allow Egress to DNS
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
- CNI that supports NetworkPolicy (Calico, Cilium, etc.)

```bash
kubectl create ns netpol-lab
kubectl config set-context --current --namespace=netpol-lab
```

---

## 2. Core Concepts
- **Default allow:** Kubernetes allows traffic unless restricted by NetworkPolicy
- **Policy selectivity:** Policies apply only to selected pods

---

## 3. Lab 1: Default Deny Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

### Apply and test
```bash
kubectl apply -f - <<'YAML'
<PASTE THE YAML ABOVE>
YAML
```

### Solution check
Pods cannot receive inbound traffic unless allowed by another policy.

---

## 4. Lab 2: Allow Same-Namespace Traffic

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namespace
spec:
  podSelector: {}
  policyTypes: ["Ingress"]
  ingress:
  - from:
    - podSelector: {}
```

### Solution check
Pods can talk to each other inside the namespace.

---

## 5. Lab 3: Allow From Specific Namespace

```bash
kubectl create ns netpol-client
kubectl label ns netpol-client purpose=client
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-client
spec:
  podSelector: {}
  policyTypes: ["Ingress"]
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          purpose: client
```

---

## 6. Lab 4: Allow Egress to DNS

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
spec:
  podSelector: {}
  policyTypes: ["Egress"]
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
```

---

## 7. Troubleshooting Scenarios
- **Policy has no effect:** CNI may not support NetworkPolicy.
- **DNS failures:** Ensure egress to CoreDNS is allowed.
- **Cross-namespace blocked:** Check namespace labels and selectors.

---

## 8. Interview Questions and Answers

### Q1: Are NetworkPolicies default-deny?
**Answer:** No. They only deny traffic to pods they select.

### Q2: How do you allow all egress but deny ingress?
**Answer:** Use policyTypes: [Ingress] only.

### Q3: Do policies apply to hostNetwork pods?
**Answer:** Generally no, hostNetwork bypasses CNI enforcement.

---

## 9. Cleanup
```bash
kubectl delete ns netpol-lab
kubectl delete ns netpol-client
```

---

## 10. Theory (Concise)
- **Policies are additive** and only apply to selected pods.
- **Selectors define scope** across pods and namespaces.
- **DNS and kube-system** are common egress exceptions.
