# Kubernetes Lab 24: TLS and cert-manager

This lab covers TLS secrets, Ingress TLS, and cert-manager workflows.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: TLS Secret from Self-Signed Cert
4. Lab 2: Ingress TLS Termination
5. Lab 3: cert-manager Issuer (If Installed)
6. Lab 4: Certificate Rotation
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
- Ingress controller for TLS termination
- Optional: cert-manager installed

```bash
kubectl create ns tls-lab
kubectl config set-context --current --namespace=tls-lab
```

---

## 2. Core Concepts
- **TLS secret:** `kubernetes.io/tls` type
- **Ingress TLS:** Termination at the edge
- **Issuer/Certificate:** cert-manager CRDs

---

## 3. Lab 1: TLS Secret from Self-Signed Cert

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=app.tls.local"

kubectl create secret tls app-tls --cert=tls.crt --key=tls.key
```

---

## 4. Lab 2: Ingress TLS Termination

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
    - app.tls.local
    secretName: app-tls
  rules:
  - host: app.tls.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app
            port:
              number: 80
```

---

## 5. Lab 3: cert-manager Issuer (If Installed)

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned
spec:
  selfSigned: {}
```

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: app-cert
spec:
  secretName: app-cert
  dnsNames:
  - app.tls.local
  issuerRef:
    name: selfsigned
    kind: Issuer
```

---

## 6. Lab 4: Certificate Rotation

```bash
kubectl delete secret app-tls
# cert-manager reissues if configured
```

---

## 7. Troubleshooting Scenarios
- **TLS errors:** Secret type or cert host mismatch.
- **Ingress 404:** Host rules not matching.
- **cert-manager stuck:** Check events and controller logs.

---

## 8. Interview Questions and Answers

### Q1: What is stored in a TLS secret?
**Answer:** A private key and certificate chain.

### Q2: Why use cert-manager?
**Answer:** Automated certificate issuance and renewal.

### Q3: Where does TLS termination happen?
**Answer:** Typically at Ingress or gateway.

---

## 9. Cleanup
```bash
kubectl delete ns tls-lab
rm -f tls.key tls.crt
```

---

## 10. Theory (Concise)
- **TLS secrets store keys and certs** securely in Kubernetes.
- **Ingress handles termination**, offloading from pods.
- **cert-manager automates renewals** and reduces outages.
