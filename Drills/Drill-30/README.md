# ডেপথ ড্রিল 30: AWS WAF Rate Limit False Positives

1️⃣ প্রোডাকশন পরিস্থিতি
Marketing campaign এ 20k rps। WAF rate rule legit traffic এর 8% block করছে।

2️⃣ প্রমাণ
```
$ aws wafv2 get-sampled-requests --scope CLOUDFRONT --rule-metric-name RateLimitRule
Action: BLOCK  RateBasedRuleName: RateLimitRule  Rate: 2100

$ aws wafv2 get-web-acl --name prod-acl --scope CLOUDFRONT | rg -n 'RateBasedStatement'
Limit: 2000
AggregateKeyType: IP

$ cdn access log | tail -n 2
403 - blocked by WAF
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Protection রেখে false positives কমাতে best adjustment কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) rate limit বাড়ানো + scope-down statement
B) WAF বন্ধ করা
C) origin capacity বাড়ানো
D) limit 1000 করা

5️⃣ শেখার লক্ষ্য (Explicit)
Rate-based rule tuning করে balance রাখা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: current limit confirm
```
aws wafv2 get-web-acl --name prod-acl --scope CLOUDFRONT
```

### Step 2: limit + scope update
```
aws wafv2 update-web-acl --name prod-acl --scope CLOUDFRONT --id ... --lock-token ...
```

### Step 3: block rate monitor
```
aws wafv2 get-sampled-requests --scope CLOUDFRONT --rule-metric-name RateLimitRule
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: limit সামান্য বাড়িয়ে, bots/paths scope-down করলে legit traffic বাঁচে।
- কেন অন্যগুলো নয়: B unsafe, C unrelated, D আরও block বাড়ায়।

### Simple Variations
- rate limit per path ভিন্ন রাখতে পারো

## Short End-to-End (খুব সহজ)
1) Check: WAF rate limit আর block rate দেখো।
2) Fix: limit বাড়িয়ে scope-down দাও।
3) Verify: legit 403 কমছে কিনা দেখো।
