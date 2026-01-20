# ডেপথ ড্রিল 46: Cascading Timeouts from Retries

1️⃣ প্রোডাকশন পরিস্থিতি
Checkout service inventory/payment কল করে। latency spike পরে retries বাড়ায় cascade failure হচ্ছে।

2️⃣ প্রমাণ
```
$ app config | rg -n 'retries|timeout'
retries: 5
timeout_ms: 500

$ inventory metrics
p95 latency: 900ms
inflight: 1200

$ payment logs | tail -n 3
context deadline exceeded
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Cascade থামাতে immediate change কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) retries কমানো + timeout বাড়ানো
B) retries বাড়ানো
C) timeout তুলে দেওয়া
D) শুধু checkout scale করা

5️⃣ শেখার লক্ষ্য (Explicit)
Retry storm চিনে timeout alignment করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: upstream latency observe
```
# inventory metrics দেখো
```

### Step 2: retry/timeout adjust
```
# app config
retries: 2
timeout_ms: 1200
```

### Step 3: error rate কমছে কিনা দেখো
- SLO/metrics check

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: কম retry + realistic timeout এ load কমে cascade থামে।
- কেন অন্যগুলো নয়: B storm বাড়ায়, C hang risk, D partial fix।

### Simple Variations
- circuit breaker যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: upstream p95 বনাম timeout তুলনা করো।
2) Fix: retries কমাও + timeout বাড়াও।
3) Verify: error rate কমছে কিনা দেখো।
