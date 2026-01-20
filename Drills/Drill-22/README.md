# ডেপথ ড্রিল 22: HPA Fails to Scale on Custom Metrics

1️⃣ প্রোডাকশন পরিস্থিতি
Queue depth দিয়ে HPA scale করে। traffic spike হলেও replicas 3-এ আটকে আছে, latency বাড়ছে।

2️⃣ প্রমাণ
```
$ kubectl describe hpa -n worker worker-hpa
Conditions:
  AbleToScale False  FailedGetExternalMetric  the server could not find the requested resource

$ kubectl get apiservices | rg external.metrics
v1beta1.external.metrics.k8s.io  False (MissingEndpoints)

$ kubectl -n monitoring get deploy metrics-adapter
0/1
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Scaling ফেরাতে সবচেয়ে immediate fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) metrics-adapter restore করা
B) maxReplicas বাড়ানো
C) CPU-based HPA সেট করা (adapter ঠিক না করে)
D) manual scale

5️⃣ শেখার লক্ষ্য (Explicit)
External metrics dependency বোঝা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: external.metrics API দেখো
```
kubectl get apiservices | rg external.metrics
```

### Step 2: adapter ঠিক করো
```
kubectl -n monitoring rollout restart deploy/metrics-adapter
```

### Step 3: HPA status verify
```
kubectl describe hpa -n worker worker-hpa
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: adapter down থাকলে HPA external metrics পায় না।
- কেন অন্যগুলো নয়: B/C/D workaround, root cause নয়।

### Simple Variations
- adapter HA করলে future outage কমবে

## Short End-to-End (খুব সহজ)
1) Check: external.metrics API missing কিনা দেখো।
2) Fix: metrics-adapter restart/restore করো।
3) Verify: HPA scale হচ্ছে কিনা দেখো।
