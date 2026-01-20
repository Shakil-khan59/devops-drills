# ডেপথ ড্রিল 36: CloudFront Cache Key Explosion

1️⃣ প্রোডাকশন পরিস্থিতি
Marketing page এ user segment header যোগ হয়েছে। cache hit 90% থেকে 30%। origin CPU saturated।

2️⃣ প্রমাণ
```
$ aws cloudfront get-distribution-config --id E456 | rg -n 'Headers'
Headers: ["*"]

$ origin logs | head -n 2
GET /promo 200 1.3s header:X-User-Segment=A
GET /promo 200 1.2s header:X-User-Segment=B

$ cloudwatch get-metric-statistics --metric-name OriginLatency
Average: 1.1
```

3️⃣ সিদ্ধান্ত প্রশ্ন
cache efficiency ফেরাতে best change কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) cache key-তে শুধু X-User-Segment whitelist
B) সব header বাদ দেওয়া
C) origin scale up
D) CloudFront বন্ধ করা

5️⃣ শেখার লক্ষ্য (Explicit)
cache key সীমিত রেখে performance ঠিক রাখা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: header-based cache key confirm
```
aws cloudfront get-distribution-config --id E456 | rg -n 'Headers'
```

### Step 2: whitelist apply
```
aws cloudfront update-distribution --id E456 --distribution-config file://dist.json
```

### Step 3: cache hit observe
```
aws cloudwatch get-metric-statistics --metric-name CacheHitRate --namespace AWS/CloudFront
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: all headers cache key explode করে; only required header রাখলে hit ratio বাড়ে।
- কেন অন্যগুলো নয়: B functionality ভাঙে, C খরচ বাড়ায়, D extreme।

### Simple Variations
- segmentation header কে query param এ আনতে পারো

## Short End-to-End (খুব সহজ)
1) Check: cache key-তে `Headers: ["*"]` আছে কিনা দেখো।
2) Fix: শুধু দরকারি header whitelist করো।
3) Verify: cache hit বাড়ছে কিনা দেখো।
