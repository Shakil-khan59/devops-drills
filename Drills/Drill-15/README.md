# ডেপথ ড্রিল 15: GKE Node Upgrade and PDB Deadlock

1️⃣ প্রোডাকশন পরিস্থিতি
GKE auto-upgrade চলাকালীন node drain fail হচ্ছে। stateful service আছে; SLO 99.95%।

2️⃣ প্রমাণ
```
$ kubectl get pdb -n search
search-pdb  minAvailable: 3  allowedDisruptions: 0

$ kubectl drain gke-node-17 --ignore-daemonsets
error: cannot evict pod as it would violate the pod's disruption budget.

$ kubectl get pods -n search -o wide | wc -l
3
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Upgrade চালাতে safest পরিবর্তন কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) replicas বাড়িয়ে PDB adjust করা
B) force delete pods
C) auto-upgrade বন্ধ করা
D) PDB মুছে ফেলা

5️⃣ শেখার লক্ষ্য (Explicit)
PDB আর maintenance balance করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: PDB block করছে কিনা দেখো
```
kubectl get pdb -n search
```

### Step 2: capacity বাড়াও
```
kubectl -n search scale deploy/search --replicas=4
```

### Step 3: PDB update
```
kubectl -n search patch pdb search-pdb -p '{"spec":{"minAvailable":3}}'
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: extra replica যোগ করে disruption allow করা safe।
- কেন অন্যগুলো নয়: B/D risky, C operational debt।

### Simple Variations
- maintenance window আগে capacity buffer রাখো

## Short End-to-End (খুব সহজ)
1) Check: PDB allowedDisruptions 0 কিনা দেখো।
2) Fix: replicas বাড়িয়ে PDB adjust করো।
3) Verify: node drain সফল হচ্ছে কিনা দেখো।
