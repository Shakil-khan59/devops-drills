# ডেপথ ড্রিল 10: Ingress 502 During Rollout

1️⃣ প্রোডাকশন পরিস্থিতি
`api` নতুন version rollout হচ্ছে (300 pods)। NGINX Ingress শুধু নতুন pods-এ 502 দিচ্ছে। rollback এড়াতে হবে।

2️⃣ প্রমাণ
```
$ kubectl logs -n ingress-nginx deploy/ingress-nginx-controller | tail -n 5
[error] 1234#1234: *9021 upstream prematurely closed connection while reading response header from upstream

$ kubectl describe ingress api | rg -n 'proxy-read-timeout'
nginx.ingress.kubernetes.io/proxy-read-timeout: "10"

$ kubectl logs deploy/api -n api | tail -n 5
INFO: warming caches (expected 20s)
INFO: starting gRPC server

$ kubectl get pod -n api api-7c9b | rg 'READY'
1/1
```

3️⃣ সিদ্ধান্ত প্রশ্ন
502 কমাতে safest পরিবর্তন কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) proxy-read-timeout বাড়ানো + readiness ঠিক করা
B) readiness বন্ধ করা
C) rollback
D) NGINX workers বাড়ানো

5️⃣ শেখার লক্ষ্য (Explicit)
warmup time এবং ingress timeout align করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: timeout mismatch বোঝো
```
kubectl describe ingress api | rg -n 'proxy-read-timeout'
```

### Step 2: timeout + readiness adjust
```
kubectl -n api annotate ingress api nginx.ingress.kubernetes.io/proxy-read-timeout="60"
```

### Step 3: rollout verify
```
kubectl -n api rollout status deploy/api
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: app warmup 20s, timeout 10s; align করলে 502 কমে।
- কেন অন্যগুলো নয়: B unsafe, C evidence mismatch, D root cause নয়।

### Simple Variations
- startupProbe যোগ করলে আরও stable হয়

## Short End-to-End (খুব সহজ)
1) Check: app warmup time বনাম ingress timeout দেখো।
2) Fix: proxy timeout বাড়াও + readiness ঠিক করো।
3) Verify: 502 কমছে কিনা দেখো।
