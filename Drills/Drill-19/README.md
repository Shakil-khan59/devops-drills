# ডেপথ ড্রিল 19: ArgoCD Drift from External Values

1️⃣ প্রোডাকশন পরিস্থিতি
ArgoCD OutOfSync দেখাচ্ছে। Git এ change নেই, কিন্তু pods restart হয়েছে।

2️⃣ প্রমাণ
```
$ argocd app diff payments
@@
-  replicas: 6
+  replicas: 10

$ kubectl get configmap -n argocd argocd-cm -o yaml | rg -n 'helm'
helm.values: "s3://config-bucket/prod/values.yaml"

$ aws s3 ls s3://config-bucket/prod/values.yaml
2024-05-11 12:05:00  2412 values.yaml
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Hidden drift ঠেকাতে সঠিক fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) values git-এ আনা এবং pin করা
B) auto-sync বন্ধ করা
C) diff ignore list বাড়ানো
D) আগের sync rollback

5️⃣ শেখার লক্ষ্য (Explicit)
GitOps-এ immutable config রাখা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: external values source confirm
```
kubectl get configmap -n argocd argocd-cm -o yaml | rg -n 'helm'
```

### Step 2: values git-এ move
```
# values.yaml git repo তে add
argocd app set payments --values values.yaml
```

### Step 3: sync verify
```
argocd app sync payments
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: external mutable values drift তৈরি করে; git pin করলে audit সম্ভব।
- কেন অন্যগুলো নয়: B/C/D drift লুকায়।

### Simple Variations
- policy হিসেবে external values নিষেধ করো

## Short End-to-End (খুব সহজ)
1) Check: external values source (S3) আছে কিনা দেখো।
2) Fix: values git-এ এনে pin করো।
3) Verify: ArgoCD sync clean কিনা দেখো।
