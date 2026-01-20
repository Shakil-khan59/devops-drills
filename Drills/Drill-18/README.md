# ডেপথ ড্রিল 18: CI/CD Concurrency Cancels Partial Deploy

1️⃣ প্রোডাকশন পরিস্থিতি
GitHub Actions concurrency `cancel-in-progress` এর কারণে deploy মাঝপথে cancel হয়েছে। half pods old, half pods new। errors বাড়ছে।

2️⃣ প্রমাণ
```
$ gh run view 123 --log | tail -n 4
job deploy cancelled
job migrate completed

$ kubectl get deploy -n api
api  3/6  image: v2.1.3

$ kubectl get job -n api | rg migrate
migrate-2024-05-11  Completed
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Consistency ফেরাতে safest action কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) deploy job আবার run করে converge করা
B) DB migration rollback
C) concurrency cancel বন্ধ করা
D) deploy scale to zero

5️⃣ শেখার লক্ষ্য (Explicit)
Partial deploy state safely recover করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: run state দেখো
```
gh run view 123 --log
```

### Step 2: deploy rerun
```
gh run rerun 123 --job deploy
```

### Step 3: rollout verify
```
kubectl -n api rollout status deploy/api
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: deploy আবার চালালে single version converge হয়।
- কেন অন্যগুলো নয়: B risky, C policy change immediate fix নয়, D disruptive।

### Simple Variations
- migration compatibility checks যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: deploy job cancelled কিনা দেখো।
2) Fix: deploy job rerun করো।
3) Verify: rollout complete কিনা দেখো।
