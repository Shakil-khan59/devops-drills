# ডেপথ ড্রিল 43: CSI Driver Crash and Mount Failures

1️⃣ প্রোডাকশন পরিস্থিতি
StatefulSet pods `ContainerCreating` এ আটকে। CSI controller CrashLoop।

2️⃣ প্রমাণ
```
$ kubectl describe pod db-2 | rg -n 'MountVolume|AttachVolume'
MountVolume.MountDevice failed for volume "pvc-123": rpc error: code = Internal desc = timeout

$ kubectl get pods -n kube-system | rg csi
csi-controller-0  0/1  CrashLoopBackOff

$ kubectl logs -n kube-system csi-controller-0 | tail -n 3
panic: nil pointer dereference
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Volume attach recover করতে safest first action কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) CSI driver rollback
B) manual attach
C) PVC delete
D) সব node restart

5️⃣ শেখার লক্ষ্য (Explicit)
CSI instability safe ভাবে ঠিক করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: CSI controller logs দেখো
```
kubectl logs -n kube-system csi-controller-0 | tail -n 10
```

### Step 2: rollback
```
kubectl -n kube-system rollout undo deploy/csi-controller
```

### Step 3: mount retry verify
```
kubectl get pods -n <ns>
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: known good version-এ ফিরলে mounts recover হয়।
- কেন অন্যগুলো নয়: B/C data risk, D disruptive।

### Simple Variations
- CSI canary rollout রাখো

## Short End-to-End (খুব সহজ)
1) Check: CSI controller CrashLoop logs দেখো।
2) Fix: driver rollback করো।
3) Verify: pod mount succeed হচ্ছে কিনা দেখো।
