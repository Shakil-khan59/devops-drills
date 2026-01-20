# ডেপথ ড্রিল 08: CoreDNS NXDOMAIN Storm

1️⃣ প্রোডাকশন পরিস্থিতি
2k-node cluster-এ DNS failure 3%। CoreDNS CPU high। 30 মিনিটে stabilize করতে হবে।

2️⃣ প্রমাণ
```
$ kubectl -n kube-system top pod | rg coredns
coredns-6b6b4c5d6d-9l6fr   460m   240Mi

$ kubectl logs -n kube-system deploy/coredns | tail -n 5
[INFO] 10.2.34.5:52418 - 33821 "A IN app.internal" NXDOMAIN 78 0.001

$ kubectl -n kube-system get configmap coredns -o yaml | rg -n 'cache|forward'
cache 30
forward . /etc/resolv.conf

$ kubectl exec -n tools dns-debug -- dig +short app.internal
<empty>
```

3️⃣ সিদ্ধান্ত প্রশ্ন
NXDOMAIN storm কমাতে best mitigation কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) CoreDNS scale + cache বাড়ানো
B) cache বন্ধ করা
C) extra search domains যোগ করা
D) বারবার CoreDNS restart

5️⃣ শেখার লক্ষ্য (Explicit)
DNS storm এ caching + scaling ব্যবহার করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: cache এবং CPU দেখো
```
kubectl -n kube-system top pod | rg coredns
```

### Step 2: cache বাড়াও + replicas বাড়াও
```
kubectl -n kube-system scale deploy/coredns --replicas=6
kubectl -n kube-system edit configmap coredns
```

### Step 3: query rate স্থির হচ্ছে কিনা দেখো
```
kubectl logs -n kube-system deploy/coredns | tail -n 5
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: repeated NXDOMAIN cache হলে load কমে, replicas বাড়লে capacity বাড়ে।
- কেন অন্যগুলো নয়: B load বাড়ায়, C irrelevant, D temporary।

### Simple Variations
- noisy client খুঁজে rate-limit করো

## Short End-to-End (খুব সহজ)
1) Check: CoreDNS CPU আর NXDOMAIN logs দেখো।
2) Fix: replicas বাড়াও + cache সময় বাড়াও।
3) Verify: DNS failure rate কমছে কিনা দেখো।
