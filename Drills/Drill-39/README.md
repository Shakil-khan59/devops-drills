# ডেপথ ড্রিল 39: Loki Ingestion Throttling

1️⃣ প্রোডাকশন পরিস্থিতি
Loki 5TB/day ingest করে। spike এ recent logs 15 মিনিট missing।

2️⃣ প্রমাণ
```
$ kubectl logs -n logging deploy/loki | tail -n 5
level=warn msg="ingestion rate limit exceeded" tenant=prod

$ kubectl get cm -n logging loki-config -o yaml | rg -n 'ingestion_rate_mb'
ingestion_rate_mb: 8

$ kubectl top pod -n logging | rg loki
CPU: 2.9 cores  Memory: 6.1Gi
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Log visibility ফেরাতে best immediate mitigation কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) ingestion limit বাড়িয়ে ingester scale করা
B) prod ছাড়া সব logs drop করা
C) Loki restart করা
D) rate limit permanently off

5️⃣ শেখার লক্ষ্য (Explicit)
Ingestion limit vs capacity balance করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: ingestion limit দেখো
```
kubectl get cm -n logging loki-config -o yaml | rg -n 'ingestion_rate_mb'
```

### Step 2: limit + scale
```
kubectl -n logging edit cm loki-config
kubectl -n logging scale deploy/loki-ingester --replicas=6
```

### Step 3: recent logs verify
```
# queries or recent log checks
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: capacity বাড়ালে rate limit hit কমে।
- কেন অন্যগুলো নয়: B data loss, C temporary, D unsafe।

### Simple Variations
- burst budget আলাদা করে রাখো

## Short End-to-End (খুব সহজ)
1) Check: ingestion limit hit হচ্ছে কিনা দেখো।
2) Fix: limit বাড়াও + ingester scale করো।
3) Verify: recent logs দেখা যাচ্ছে কিনা দেখো।
