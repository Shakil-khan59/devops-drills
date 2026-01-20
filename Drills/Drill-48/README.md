# ডেপথ ড্রিল 48: Canary Metrics Misattribution

1️⃣ প্রোডাকশন পরিস্থিতি
Canary error-rate 2% দেখিয়ে auto rollback হয়েছে। পরে দেখা গেল metric misattributed।

2️⃣ প্রমাণ
```
$ promql
sum(rate(http_requests_total{job="api"}[5m])) by (status)

$ canary config
analysis:
  metrics:
  - name: error-rate
    query: sum(rate(http_requests_total{job="api"}[1m]))
    threshold: 0.01

$ labels on canary pods
version=canary
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Canary metric isolate করার best change কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) canary label দিয়ে query filter করা
B) cluster-wide error rate ব্যবহার করা
C) threshold 5% করা
D) auto rollback বন্ধ করা

5️⃣ শেখার লক্ষ্য (Explicit)
Canary analysis metrics scope করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: metrics query scope check
```
# current query lacks version label
```

### Step 2: label filter যোগ করো
```
sum(rate(http_requests_total{job="api",version="canary"}[1m])) by (status)
```

### Step 3: baseline compare
- canary vs stable error rate compare

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: canary-only metrics দিলে misattribution হয় না।
- কেন অন্যগুলো নয়: B noisy, C hides issue, D unsafe।

### Simple Variations
- canary traffic separate service দিয়ে পাঠাও

## Short End-to-End (খুব সহজ)
1) Check: canary label query-তে আছে কিনা দেখো।
2) Fix: query-তে `version=canary` filter দাও।
3) Verify: canary-only error rate দেখো।
