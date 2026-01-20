# ডেপথ ড্রিল 40: Prometheus Federation Data Gaps

1️⃣ প্রোডাকশন পরিস্থিতি
Global dashboard federated Prometheus থেকে data নেয়। peak-এ 10 মিনিট gap হচ্ছে।

2️⃣ প্রমাণ
```
$ curl -s http://federator:9090/metrics | rg 'prometheus_remote_storage_queue'
prometheus_remote_storage_queue_highest_sent_timestamp_seconds 1715430000

$ curl -s http://region-a:9090/api/v1/targets | rg -n 'lastScrape'
lastScrape: "2024-05-11T12:00:02Z"

$ kubectl top pod -n monitoring | rg federator
CPU: 3.8 cores  Memory: 4.2Gi
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Gap কমাতে best fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) federation scrape interval বাড়ানো + target set কমানো
B) scrape interval 1s করা
C) federation বন্ধ করা
D) সব high-cardinality metrics drop করা

5️⃣ শেখার লক্ষ্য (Explicit)
Federation scale করতে cardinality control করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: federator load দেখো
```
kubectl top pod -n monitoring | rg federator
```

### Step 2: scrape scope কমাও
```
# federator config এ critical metrics রাখো
kubectl -n monitoring edit cm federator-config
```

### Step 3: gap কমছে কিনা দেখো
```
# dashboard / metrics check
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: কম target + বড় interval এ load কমে gap কমে।
- কেন অন্যগুলো নয়: B load বাড়ায়, C visibility হারায়, D too blunt।

### Simple Variations
- remote_write + downsampling ব্যবহার করো

## Short End-to-End (খুব সহজ)
1) Check: federator CPU high কিনা দেখো।
2) Fix: scrape target কমাও + interval বাড়াও।
3) Verify: data gaps কমছে কিনা দেখো।
