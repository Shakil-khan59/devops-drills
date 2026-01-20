# ডেপথ ড্রিল 45: Secrets Rotation Breaks Auth

1️⃣ প্রোডাকশন পরিস্থিতি
OAuth secret rotation এর পরে 30% requests 401। security বন্ধ করা যাবে না।

2️⃣ প্রমাণ
```
$ kubectl get secret -n api oauth-client -o jsonpath='{.metadata.annotations}'
{"rotation":"2024-05-11T12:10:00Z"}

$ app logs | tail -n 4
OAuth token exchange failed: invalid_client

$ kubectl rollout status deploy/api -n api
deployment "api" successfully rolled out
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Auth restore করতে best action কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) dual secrets support করে gradual rotate
B) পুরনো secret permanently ফিরিয়ে আনা
C) OAuth validation বন্ধ করা
D) pods restart loop

5️⃣ শেখার লক্ষ্য (Explicit)
Secret rotation এ overlap strategy বোঝা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: rotation timestamp confirm
```
kubectl get secret -n api oauth-client -o jsonpath='{.metadata.annotations}'
```

### Step 2: dual secret support
- old + new secret একসাথে allow করা

### Step 3: gradual rollout
```
kubectl -n api rollout restart deploy/api
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: overlap থাকলে clients ধীরে migrate হয়।
- কেন অন্যগুলো নয়: B blocks rotation, C insecure, D unreliable।

### Simple Variations
- rotation window policy যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: rotation timestamp আর 401 logs দেখো।
2) Fix: old+new secret overlap দিয়ে rollout করো।
3) Verify: auth failures কমছে কিনা দেখো।
