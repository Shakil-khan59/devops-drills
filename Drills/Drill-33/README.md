# ডেপথ ড্রিল 33: GitOps Rollback Fails on Immutable Field

1️⃣ প্রোডাকশন পরিস্থিতি
ArgoCD rollback সময় `spec.selector` immutable error দিচ্ছে। Error budget burn হচ্ছে।

2️⃣ প্রমাণ
```
$ argocd app sync api
Sync failed: Deployment.apps "api" is invalid: spec.selector: Invalid value: field is immutable

$ kubectl get deploy api -o yaml | rg -n 'selector'
selector:
  matchLabels:
    app: api
    version: v2

$ git show HEAD~1:deploy/api.yaml | rg -n 'selector'
app: api
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Availability রেখে safe rollback path কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) পুরনো selector দিয়ে নতুন Deployment করে traffic shift
B) `kubectl replace --force`
C) Deployment delete করে recreate
D) rollback না করা

5️⃣ শেখার লক্ষ্য (Explicit)
Immutable field rollback safely handle করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: selector mismatch confirm
```
kubectl get deploy api -o yaml | rg -n 'selector'
```

### Step 2: নতুন Deployment তৈরি
```
kubectl apply -f deploy-api-v1.yaml
```

### Step 3: traffic shift
```
kubectl patch svc api -p '{"spec":{"selector":{"app":"api","version":"v1"}}}'
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: immutable selector এ in-place rollback risky; new deploy safer।
- কেন অন্যগুলো নয়: B/C downtime risk, D ignores incident।

### Simple Variations
- future-এ selector পরিবর্তন না করার policy রাখো

## Short End-to-End (খুব সহজ)
1) Check: selector immutable error দেখো।
2) Fix: old selector দিয়ে নতুন Deployment বানাও।
3) Verify: service selector switch হয়ে traffic ঠিক আছে কিনা দেখো।
