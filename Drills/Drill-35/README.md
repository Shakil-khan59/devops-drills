# ডেপথ ড্রিল 35: KMS Throttling Causes Auth Latency

1️⃣ প্রোডাকশন পরিস্থিতি
প্রতি request এ KMS Decrypt করলে p95 latency 80ms থেকে 500ms। Throttling বাড়ছে।

2️⃣ প্রমাণ
```
$ aws cloudwatch get-metric-statistics --namespace AWS/KMS --metric-name Throttles
Average: 12

$ app logs | tail -n 3
KMS Decrypt latency=420ms
KMS Decrypt error: ThrottlingException

$ grep -R "decrypt" app/config.yml
decrypt_on_request: true
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Security রেখে latency কমাতে best mitigation কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) decrypted secrets cache করা (TTL সহ)
B) KMS policy খুলে দেওয়া
C) plaintext secrets রাখা
D) timeout বাড়ানো

5️⃣ শেখার লক্ষ্য (Explicit)
KMS usage pattern ঠিক করে latency কমানো।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: per-request decrypt আছে কিনা দেখো
```
grep -R "decrypt" app/config.yml
```

### Step 2: cache enable
```
# app config
cache_ttl_seconds: 300
```

### Step 3: throttles monitor
```
aws cloudwatch get-metric-statistics --namespace AWS/KMS --metric-name Throttles
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: cache করলে KMS calls কমে এবং latency কমে।
- কেন অন্যগুলো নয়: B/C security risk, D symptom ঢাকে।

### Simple Variations
- envelope encryption ব্যবহার করো

## Short End-to-End (খুব সহজ)
1) Check: per-request decrypt config আছে কিনা দেখো।
2) Fix: decrypted secrets cache করো।
3) Verify: KMS throttles কমছে কিনা দেখো।
