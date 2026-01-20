# ডেপথ ড্রিল 02: Service Mesh Sidecar OOM Spiral

1️⃣ প্রোডাকশন পরিস্থিতি
400-service Istio mesh-এ traffic spike এর পরে 503 burst হচ্ছে। Error budget 6x burn। rollback এড়াতে হবে।

2️⃣ প্রমাণ
```
$ kubectl get pods -n payments | rg 'CrashLoop|OOM'
api-7f7dd5d8f9-9q9kz   1/2   OOMKilled   4   12m

$ kubectl logs api-7f7dd5d8f9-9q9kz -c istio-proxy | tail -n 5
[warning][memory] Memory pressure high. Allocated 498Mi, limit 512Mi
[warning][stats] flushing stats took 280ms

$ istioctl proxy-config bootstrap api-7f7dd5d8f9-9q9kz | rg 'stats'
"stats_flush_interval": "1s"

$ kubectl describe pod api-7f7dd5d8f9-9q9kz | rg -n 'Limits|Requests'
Limits:  cpu: 500m  memory: 512Mi
Requests: cpu: 100m  memory: 128Mi
```

3️⃣ সিদ্ধান্ত প্রশ্ন
কোন action ট্রাফিক stabilize করবে এবং root cause ঠিক করবে?

4️⃣ সম্ভাব্য পথ (3–6)
A) শুধু app container memory বাড়ানো
B) sidecar memory বাড়ানো + stats flush interval বড় করা
C) global mTLS বন্ধ করা
D) rollout rollback করা

5️⃣ শেখার লক্ষ্য (Explicit)
sidecar resource pressure বোঝা এবং নিরাপদ mitigation নেওয়া।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough (End-to-End)

### Step 1: OOM কোন container-এ দেখো
```
kubectl describe pod api-7f7dd5d8f9-9q9kz
```

### Step 2: sidecar sizing ঠিক করো
```
kubectl -n payments set resources deploy/api -c istio-proxy --limits=memory=1Gi --requests=memory=256Mi
kubectl -n payments patch deploy/api -p '{"spec":{"template":{"metadata":{"annotations":{"proxy.istio.io/config":"{\"stats_flush_interval\":\"10s\"}"}}}}}'
```

### Step 3: error rate কমছে কি দেখো
```
kubectl -n payments get pods
```

---

## Solution (Simple)
- Chosen option(s): B
- কেন ঠিক: evidence অনুযায়ী istio-proxy OOM হচ্ছে; memory + flush interval ঠিক করলে overhead কমে।
- কেন অন্যগুলো নয়: A root cause মিস করে, C security ভাঙে, D evidence-এর সাথে মেলে না।

### দ্রুত যাচাই
```
kubectl -n payments get pods
kubectl -n payments logs deploy/api -c istio-proxy | tail -n 5
```

### Simple Variations
- CPU spike হলে telemetry sampling কমাও
- cert error হলে SDS/rotation চেক করো

## Short End-to-End (খুব সহজ)
1) Check: pod describe/logs-এ `istio-proxy` OOM আছে কিনা দেখো।
2) Fix: sidecar memory বাড়াও + stats flush interval বাড়াও।
3) Verify: OOM stop এবং 503 কমছে কিনা দেখো।
