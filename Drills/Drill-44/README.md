# ডেপথ ড্রিল 44: Overprovisioning from Missing Requests

1️⃣ প্রোডাকশন পরিস্থিতি
Karpenter cluster 40 থেকে 200 nodes এ scale হয়েছে, traffic unchanged। cost 3x।

2️⃣ প্রমাণ
```
$ kubectl get pods -A -o custom-columns=NAME:.metadata.name,CPU:.spec.containers[*].resources.requests.cpu | head -n 3
api-7f9d   <none>
worker-6c8  <none>

$ kubectl -n kube-system logs deploy/karpenter | tail -n 3
provisioning new node: reason=unschedulable

$ kubectl get nodes | wc -l
200
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Root cause fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) realistic CPU/memory requests set করা
B) Karpenter disable
C) max nodes 50 করা
D) pod limits বাড়ানো

5️⃣ শেখার লক্ষ্য (Explicit)
Requests না দিলে overprovision কেন হয় বোঝা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: requests missing কিনা দেখো
```
kubectl get pods -A -o custom-columns=NAME:.metadata.name,CPU:.spec.containers[*].resources.requests.cpu | head -n 5
```

### Step 2: requests add করো
```
kubectl -n api set resources deploy/api --requests=cpu=500m,memory=512Mi
```

### Step 3: scale-down observe
```
kubectl get nodes | wc -l
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: scheduler request দেখে capacity ঠিক করে; missing হলে over-scale হয়।
- কেন অন্যগুলো নয়: B/C/D workaround।

### Simple Variations
- VPA recommendations যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: pods-এ requests missing কিনা দেখো।
2) Fix: realistic requests সেট করো।
3) Verify: node count ধীরে কমছে কিনা দেখো।
