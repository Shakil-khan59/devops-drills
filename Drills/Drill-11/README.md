# ডেপথ ড্রিল 11: Route53 Health Check Flapping

1️⃣ প্রোডাকশন পরিস্থিতি
Multi-region active-active API (us-east-1, eu-west-1)। Route53 latency routing আছে। 15% client 5xx পাচ্ছে কারণ health check flapping।

2️⃣ প্রমাণ
```
$ aws route53 get-health-check-status --health-check-id 123
StatusReport: Status: "HealthCheckFailure"  CheckedTime: 2024-05-11T12:03:44Z

$ aws route53 get-health-check --health-check-id 123
HealthCheckConfig: {"Type":"HTTP","ResourcePath":"/health","RequestInterval":30,"FailureThreshold":2}

$ curl -s -o /dev/null -w "%{http_code}\n" https://api.example.com/health
200

$ curl -s -o /dev/null -w "%{http_code}\n" https://lb-eu.example.com/health
503
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Flapping কমাতে সবচেয়ে safe first step কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) failure threshold বাড়িয়ে region health ঠিক করা
B) health check তুলে দেওয়া
C) TTL 5s করা
D) সব ট্রাফিক us-east-1 এ পাঠানো

5️⃣ শেখার লক্ষ্য (Explicit)
DNS health check স্থিতিশীল করা এবং region-specific health যাচাই করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: কোন region fail করছে দেখো
```
curl -s -o /dev/null -w "%{http_code}\n" https://lb-eu.example.com/health
```

### Step 2: failure threshold adjust
```
aws route53 update-health-check --health-check-id 123 --failure-threshold 3
```

### Step 3: region LB fix
- eu-west health endpoint stable করো

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: root cause হলো region-specific health; threshold সামান্য বাড়ালে flapping কমে।
- কেন অন্যগুলো নয়: B unsafe, C unrelated, D overreaction।

### Simple Variations
- health endpoint lightweight রাখো যাতে quick response দেয়

## Short End-to-End (খুব সহজ)
1) Check: কোন region 503 দিচ্ছে তা দেখো।
2) Fix: failure threshold বাড়াও + region health ঠিক করো।
3) Verify: health checks stable কিনা দেখো।
