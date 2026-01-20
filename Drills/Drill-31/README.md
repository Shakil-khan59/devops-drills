# ডেপথ ড্রিল 31: Istio Config Push CPU Spike

1️⃣ প্রোডাকশন পরিস্থিতি
প্রতি Istio config change এ সব pods-এ 20% CPU spike হয়, latency কমে না। blast radius কমাতে হবে।

2️⃣ প্রমাণ
```
$ istioctl proxy-status | head -n 3
NAME                                CDS  LDS  EDS  RDS  ECDS  ISTIOD
api-7f9d...                         SYNCED

$ kubectl logs -n istio-system deploy/istiod | tail -n 5
ADS: Push debounce stable 1s, full push size=48000

$ kubectl top pod -n api | head -n 2
api-7f9d...  780m  600Mi
```

3️⃣ সিদ্ধান্ত প্রশ্ন
CPU spike কমাতে সবচেয়ে practical fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) config scope কমিয়ে exportTo/workloadSelector ব্যবহার
B) static Envoy config ব্যবহার করা
C) শুধু istiod replicas বাড়ানো
D) telemetry বন্ধ করা

5️⃣ শেখার লক্ষ্য (Explicit)
control-plane blast radius কমানোর পদ্ধতি শেখা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: full push হচ্ছে কিনা দেখো
```
kubectl logs -n istio-system deploy/istiod | tail -n 20
```

### Step 2: config scope কমাও
```
# DestinationRule/VirtualService এ exportTo/workloadSelector যোগ
kubectl apply -f scoped-config.yaml
```

### Step 3: CPU spike observe
```
kubectl top pod -n api | head -n 5
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: full push ছোট করলে প্রতিটি change-এ কম workload impact হয়।
- কেন অন্যগুলো নয়: B loses flexibility, C symptom only, D monitoring হারায়।

### Simple Variations
- config churn কমাতে batching/policy যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: Istio full push size বড় কিনা দেখো।
2) Fix: config scope কমাও (exportTo/workloadSelector)।
3) Verify: CPU spike কমছে কিনা দেখো।
