# ডেপথ ড্রিল 03: ALB 5xx After Path Change

1️⃣ প্রোডাকশন পরিস্থিতি
AWS ALB-এর পিছনে public API (15k rps) চলছে। `/healthz` path change করার পর শুধু new tasks-এ 5xx spike হচ্ছে। Deployment থামানো যাবে না।

2️⃣ প্রমাণ
```
$ aws elbv2 describe-target-health --target-group-arn arn:aws:elasticloadbalancing:...:targetgroup/api
TargetHealth: State: unhealthy  Reason: Health checks failed  Description: Health checks failed with these codes: [404]

$ aws elbv2 describe-target-groups --names api
HealthCheckPath: /healthz
HealthCheckPort: traffic-port

$ curl -s -o /dev/null -w "%{http_code}\n" http://10.2.14.8:8080/healthz
404

$ curl -s -o /dev/null -w "%{http_code}\n" http://10.2.14.8:8080/health
200
```

3️⃣ সিদ্ধান্ত প্রশ্ন
সবচেয়ে safe fix কোনটা, যাতে health checks ঠিক থাকে?

4️⃣ সম্ভাব্য পথ (3–6)
A) ALB health check path আবার `/health` করা
B) unhealthy threshold কমানো
C) health check bypass rule দেওয়া
D) health check সাময়িক বন্ধ করা

5️⃣ শেখার লক্ষ্য (Explicit)
health check path mismatch খুঁজে minimal-risk fix করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: health check path confirm
```
aws elbv2 describe-target-groups --names api
```

### Step 2: path ঠিক করো
```
aws elbv2 modify-target-group --target-group-arn arn:... --health-check-path /health
```

### Step 3: target healthy হয়েছে কি দেখো
```
aws elbv2 describe-target-health --target-group-arn arn:...
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: app-এ `/health` আছে, `/healthz` নেই; path ঠিক করলেই fix হয়।
- কেন অন্যগুলো নয়: B/C/D health safety কমায়।

### Simple Variations
- যদি app `/healthz` যোগ করে, তখন ALB path বদলানো লাগবে না

## Short End-to-End (খুব সহজ)
1) Check: ALB health path আর app endpoint একই কিনা দেখো।
2) Fix: health check path ঠিক করো।
3) Verify: target health `healthy` হচ্ছে কিনা দেখো।
