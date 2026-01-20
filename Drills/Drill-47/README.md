# ডেপথ ড্রিল 47: gRPC Load Balancing Skew

1️⃣ প্রোডাকশন পরিস্থিতি
gRPC service-এ 10 pods। একটি pod overloaded, অন্যরা idle। p99 2s।

2️⃣ প্রমাণ
```
$ kubectl top pod -n api | head -n 3
api-1  CPU: 900m  Memory: 1.2Gi
api-2  CPU: 120m  Memory: 400Mi

$ app config | rg -n 'load_balancing_policy'
load_balancing_policy: pick_first

$ grpcurl -plaintext api:50051 list
OK
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Load balance ঠিক করতে best fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) round_robin বা xDS LB চালু করা
B) hot pod CPU বাড়ানো
C) replica কমানো
D) gRPC বাদ দেওয়া

5️⃣ শেখার লক্ষ্য (Explicit)
Client-side gRPC LB policy বোঝা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: LB policy check
```
# app config
load_balancing_policy: pick_first
```

### Step 2: policy update
```
load_balancing_policy: round_robin
```

### Step 3: load distribution দেখো
```
kubectl top pod -n api | head -n 5
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: pick_first এক pod-এ pin করে; round_robin evenly distribute করে।
- কেন অন্যগুলো নয়: B masks, C wastes capacity, D unnecessary।

### Simple Variations
- service mesh xDS করলে dynamic LB হয়

## Short End-to-End (খুব সহজ)
1) Check: LB policy `pick_first` কিনা দেখো।
2) Fix: `round_robin` বা xDS enable করো।
3) Verify: pod CPU সমান হচ্ছে কিনা দেখো।
