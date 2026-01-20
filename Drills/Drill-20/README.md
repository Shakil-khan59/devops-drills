# ডেপথ ড্রিল 20: mTLS Handshake Failures After Cert Rotation

1️⃣ প্রোডাকশন পরিস্থিতি
Service mesh-এ cert rotation এর পরে 2% requests 503। Blast radius বাড়ছে।

2️⃣ প্রমাণ
```
$ kubectl logs -n payments deploy/api -c istio-proxy | tail -n 5
TLS error: 268435581:SSL routines:OPENSSL_internal:CERTIFICATE_VERIFY_FAILED

$ istioctl proxy-status | rg SYNCED
api-7f9...  SYNCED

$ kubectl get secret -n istio-system istio-ca-secret -o jsonpath='{.metadata.creationTimestamp}'
2024-05-11T11:00:00Z
```

3️⃣ সিদ্ধান্ত প্রশ্ন
mTLS নিরাপদ রেখে ট্রাফিক stabilise কীভাবে করবে?

4️⃣ সম্ভাব্য পথ (3–6)
A) cert chain propagate করে SDS refresh / proxy restart
B) mTLS বন্ধ করা
C) পুরনো cert-এ rollback
D) client timeout বাড়ানো

5️⃣ শেখার লক্ষ্য (Explicit)
cert rotation এর পরে trust chain সমস্যা debug করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: proxy logs দেখো
```
kubectl logs -n payments deploy/api -c istio-proxy | tail -n 5
```

### Step 2: secret update verify
```
kubectl get secret -n istio-system istio-ca-secret -o jsonpath='{.metadata.creationTimestamp}'
```

### Step 3: SDS refresh / restart
```
istioctl proxy-config secret deploy/api -n payments
kubectl -n payments rollout restart deploy/api
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: trust chain refresh না হলে verify fail হয়; proxy refresh করলে ঠিক হয়।
- কেন অন্যগুলো নয়: B security ভাঙে, C risky, D symptom ঢাকে।

### Simple Variations
- staged rotation করলে blast radius কমে

## Short End-to-End (খুব সহজ)
1) Check: proxy logs-এ CERTIFICATE_VERIFY_FAILED আছে কিনা দেখো।
2) Fix: SDS refresh বা proxy restart করো।
3) Verify: 503 rate কমছে কিনা দেখো।
