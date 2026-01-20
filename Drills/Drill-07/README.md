# ডেপথ ড্রিল 07: Cluster Autoscaler Stuck on PDB

1️⃣ প্রোডাকশন পরিস্থিতি
Cluster Autoscaler চালু আছে কিন্তু 300 pods Pending। scale-up হচ্ছে না।

2️⃣ প্রমাণ
```
$ kubectl get pods -n checkout | rg Pending | head -n 3
checkout-7f9d8c7c9b-bfdxl   0/1   Pending   0   18m

$ kubectl describe pod checkout-7f9d8c7c9b-bfdxl | rg -n 'FailedScheduling|Insufficient'
0/48 nodes are available: 48 Insufficient cpu.

$ kubectl -n kube-system logs deploy/cluster-autoscaler | tail -n 5
scale-up failed: pod didn't trigger scale-up (it wouldn't fit if a new node is added)

$ kubectl get pdb -n checkout
NAME            MIN AVAILABLE   ALLOWED DISRUPTIONS
checkout-pdb    28              0
```

3️⃣ সিদ্ধান্ত প্রশ্ন
স্কেলিং চালু রাখতে প্রথমে কী করবে?

4️⃣ সম্ভাব্য পথ (3–6)
A) PDB কমানো
B) চলমান pod force delete
C) বড় node size (instance type) দেওয়া
D) autoscaler বন্ধ করে manual scale

5️⃣ শেখার লক্ষ্য (Explicit)
PDB ও pod fit সমস্যা বুঝে safe scaling নেওয়া।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: pod fit check
```
kubectl describe pod checkout-7f9d8c7c9b-bfdxl
```

### Step 2: বড় node pool যোগ করো
- নতুন node group with larger instance size

### Step 3: scale-up confirm
```
kubectl get nodes
```

---

## Solution (Simple)
- Chosen option(s): C
- কেন ঠিক: autoscaler বলছে pod নতুন node-এ fit হবে না; বড় node দরকার।
- কেন অন্যগুলো নয়: A availability কমায়, B ঝুঁকিপূর্ণ, D automation হারায়।

### Simple Variations
- resource requests ঠিক থাকছে কিনা যাচাই করো

## Short End-to-End (খুব সহজ)
1) Check: Pending pod এ “wouldn’t fit” message আছে কিনা দেখো।
2) Fix: বড় node pool যোগ করো।
3) Verify: Pending pods চলা শুরু করেছে কিনা দেখো।
