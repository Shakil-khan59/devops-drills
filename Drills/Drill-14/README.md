# ডেপথ ড্রিল 14: ECS Deployment Stuck on IAM

1️⃣ প্রোডাকশন পরিস্থিতি
ECS service deployment 50% এ আটকে আছে। tasks চলছে কিন্তু ALB healthy হয় না।

2️⃣ প্রমাণ
```
$ aws ecs describe-services --cluster prod --services api | rg -n 'events' -A 3
(service api) (task 9b12...) unable to register targets with target group
AccessDenied: elasticloadbalancing:RegisterTargets

$ aws iam get-role-policy --role-name ecsTaskExecutionRole --policy-name ecs-tg
"elasticloadbalancing:RegisterTargets" not present

$ aws elbv2 describe-target-health --target-group-arn arn:...:targetgroup/api
TargetHealth: State: unused
```

3️⃣ সিদ্ধান্ত প্রশ্ন
সবচেয়ে small blast-radius fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) missing ELB permission add করা
B) health check বন্ধ করা
C) rollback
D) admin permission দেওয়া

5️⃣ শেখার লক্ষ্য (Explicit)
IAM error থেকে deployment blockage বের করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: service events দেখো
```
aws ecs describe-services --cluster prod --services api
```

### Step 2: role policy ঠিক করো
```
aws iam put-role-policy --role-name ecsTaskExecutionRole --policy-name ecs-tg --policy-document file://elbv2-register.json
```

### Step 3: target health verify
```
aws elbv2 describe-target-health --target-group-arn arn:...
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: RegisterTargets permission missing ছিল।
- কেন অন্যগুলো নয়: B/C/D unnecessary risk।

### Simple Variations
- IAM policy linting CI তে যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: ECS events-এ RegisterTargets error আছে কিনা দেখো।
2) Fix: task role-এ ELB permission যোগ করো।
3) Verify: target health `healthy` দেখো।
