# ডেপথ ড্রিল 34: API Gateway Rate Limit Misfire

1️⃣ প্রোডাকশন পরিস্থিতি
API Gateway quota change এর পরে low-traffic clients-ও 429 পাচ্ছে। support tickets বাড়ছে।

2️⃣ প্রমাণ
```
$ aws apigateway get-usage-plans --query 'items[0].throttle'
{"rateLimit": 5, "burstLimit": 10}

$ curl -i https://api.example.com/v1/orders
HTTP/1.1 429 Too Many Requests

$ cloudwatch logs | tail -n 2
Execution failed due to configuration error: Limit of 5 exceeded
```

3️⃣ সিদ্ধান্ত প্রশ্ন
সবচেয়ে reliable fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) আগের rate/burst restore + usage plan verify
B) throttling বন্ধ করা
C) burst 1000 করা
D) API key তুলে দেওয়া

5️⃣ শেখার লক্ষ্য (Explicit)
API throttling setting ঠিকমতো align করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: current throttle দেখো
```
aws apigateway get-usage-plans --query 'items[0].throttle'
```

### Step 2: rate/burst update
```
aws apigateway update-usage-plan --usage-plan-id ... --patch-operations op=replace,path=/throttle/rateLimit,value=100
```

### Step 3: client test
```
curl -i https://api.example.com/v1/orders
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: ভুল limit reset করলে legitimate traffic pass করে।
- কেন অন্যগুলো নয়: B unsafe, C imbalance, D auth ভাঙে।

### Simple Variations
- per-client quota policy যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: usage plan rate/burst দেখো।
2) Fix: আগের limits restore করো।
3) Verify: 429 কমছে কিনা দেখো।
