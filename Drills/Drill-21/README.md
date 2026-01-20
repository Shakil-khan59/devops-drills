# ডেপথ ড্রিল 21: CDN Origin 5xx Surge

1️⃣ প্রোডাকশন পরিস্থিতি
CloudFront 60% ট্রাফিক সার্ভ করছে। নতুন query param rollout এর পরে cache hit 85% থেকে 40% এ নেমে গেছে, origin 5xx বেড়েছে।

2️⃣ প্রমাণ
```
$ aws cloudwatch get-metric-statistics --metric-name CacheHitRate --namespace AWS/CloudFront
Average: 0.41

$ aws cloudfront get-distribution-config --id E123 | rg -n 'QueryString'
QueryString: true

$ curl -I 'https://cdn.example.com/page?ref=abc'
X-Cache: Miss from cloudfront

$ origin logs | tail -n 2
GET /page?ref=abc 200 1.2s
GET /page?ref=def 200 1.1s
```

3️⃣ সিদ্ধান্ত প্রশ্ন
cache efficiency ফেরাতে safest fix কী, tracking না ভেঙে?

4️⃣ সম্ভাব্য পথ (3–6)
A) দরকারি query param whitelist করা
B) query string একদম বাদ দেওয়া
C) origin capacity বাড়ানো
D) hourly invalidation

5️⃣ শেখার লক্ষ্য (Explicit)
CDN cache key control করে performance ফিরিয়ে আনা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: cache key config দেখো
```
aws cloudfront get-distribution-config --id E123
```

### Step 2: whitelist set করা
```
# cache policy update করে শুধু দরকারি params রাখা
aws cloudfront update-distribution --id E123 --distribution-config file://dist.json
```

### Step 3: hit ratio observe
```
aws cloudwatch get-metric-statistics --metric-name CacheHitRate --namespace AWS/CloudFront
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: অপ্রয়োজনীয় query param cache key explode করে; whitelist করলে hit ratio বাড়ে।
- কেন অন্যগুলো নয়: B functionality ভাঙে, C খরচ বাড়ায়, D cache churn।

### Simple Variations
- marketing param আলাদা header-এ পাঠাও

## Short End-to-End (খুব সহজ)
1) Check: cache hit drop আর query string enabled আছে কিনা দেখো।
2) Fix: শুধু দরকারি query params whitelist করো।
3) Verify: CacheHitRate বাড়ছে কিনা দেখো।
