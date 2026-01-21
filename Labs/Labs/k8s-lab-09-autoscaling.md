# Kubernetes Lab 09: Autoscaling (HPA and VPA)

This lab covers horizontal and vertical scaling for real workloads.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: Deploy a Load-Testable App
4. Lab 2: Horizontal Pod Autoscaler
5. Lab 3: Generate Load
6. Lab 4: Vertical Pod Autoscaler (If Installed)
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
- Metrics Server for HPA
- VPA controller (optional)

```bash
kubectl create ns autoscale-lab
kubectl config set-context --current --namespace=autoscale-lab
```

---

## 2. Core Concepts
- **HPA:** Scales replicas based on metrics
- **VPA:** Recommends or applies resource changes

---

## 3. Lab 1: Deploy a Load-Testable App

```bash
kubectl apply -f - <<'YAML'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cpu-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cpu-app
  template:
    metadata:
      labels:
        app: cpu-app
    spec:
      containers:
      - name: app
        image: k8s.gcr.io/hpa-example
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "500m"
        ports:
        - containerPort: 80
YAML
```

---

## 4. Lab 2: Horizontal Pod Autoscaler

```bash
kubectl autoscale deploy cpu-app --cpu-percent=50 --min=1 --max=5
kubectl get hpa
```

---

## 5. Lab 3: Generate Load

```bash
kubectl run -it --rm load --image=busybox:1.36 --restart=Never -- \
  /bin/sh -c "while true; do wget -q -O- http://cpu-app; done"
```

### Solution check
`kubectl get hpa -w` shows replica increase.

---

## 6. Lab 4: Vertical Pod Autoscaler (If Installed)

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: cpu-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cpu-app
  updatePolicy:
    updateMode: "Off"
```

### Solution check
`kubectl describe vpa cpu-app-vpa` shows recommendations.

---

## 7. Troubleshooting Scenarios
- **HPA stuck:** Metrics server not installed or not returning CPU.
- **No scale up:** Requests missing or too high.
- **VPA not working:** CRDs/controller not installed.

---

## 8. Interview Questions and Answers

### Q1: HPA vs VPA?
**Answer:** HPA changes replica count; VPA changes pod resources.

### Q2: Why set CPU requests for HPA?
**Answer:** HPA uses CPU utilization as a percentage of request.

### Q3: Can HPA and VPA work together?
**Answer:** Yes, with careful configuration; avoid conflicting updates.

---

## 9. Cleanup
```bash
kubectl delete ns autoscale-lab
```

---

## 10. Theory (Concise)
- **HPA scales horizontally** using metrics APIs.
- **VPA optimizes resource sizing** to reduce waste or throttling.
- **Requests are essential** for meaningful autoscaling.
