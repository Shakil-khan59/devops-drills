# ডেপথ ড্রিল 38: Dual-Stack IPv6 Blackhole

1️⃣ প্রোডাকশন পরিস্থিতি
Kubernetes dual-stack চালু। IPv6 clients 40% timeout, IPv4 ঠিক আছে।

2️⃣ প্রমাণ
```
$ kubectl get svc api -o yaml | rg -n 'ipFamilies|clusterIPs'
ipFamilies:
- IPv6
- IPv4
clusterIPs:
- fd00::123
- 10.3.4.5

$ ping6 fd00::123
connect: Network is unreachable

$ ip -6 route
<empty>
```

3️⃣ সিদ্ধান্ত প্রশ্ন
IPv6 connectivity ফেরাতে first action কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) nodes-এ IPv6 routes + CNI support ঠিক করা
B) IPv6 disable করা কিন্তু dual-stack রেখে দেওয়া
C) IPv6 MTU কমানো
D) replicas বাড়ানো

5️⃣ শেখার লক্ষ্য (Explicit)
Dual-stack আগে end-to-end routing verify করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: route table দেখো
```
ip -6 route
```

### Step 2: CNI/OS IPv6 config ঠিক করো
- CNI config আপডেট

### Step 3: ping6 test
```
ping6 fd00::123
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: IPv6 route না থাকলে traffic blackhole হয়।
- কেন অন্যগুলো নয়: B inconsistent, C random, D unrelated।

### Simple Variations
- region-by-region IPv6 rollout করো

## Short End-to-End (খুব সহজ)
1) Check: `ip -6 route` empty কিনা দেখো।
2) Fix: CNI/OS IPv6 routes ঠিক করো।
3) Verify: `ping6` কাজ করছে কিনা দেখো।
