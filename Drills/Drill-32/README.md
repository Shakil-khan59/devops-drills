# ডেপথ ড্রিল 32: Image Pull Failures from Registry Limits

1️⃣ প্রোডাকশন পরিস্থিতি
Node pool 50 থেকে 200 scale হওয়ার সময় pods ImagePullBackOff। registry rate limit hit করছে।

2️⃣ প্রমাণ
```
$ kubectl describe pod api-7f9d -n api | rg -n 'Failed|pull'
Failed to pull image "docker.io/org/api:1.2.3": toomanyrequests: rate limit exceeded

$ journalctl -u kubelet | tail -n 4
Error: ImagePullBackOff

$ kubectl get secret -n api | rg regcred
<none>
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Immediate mitigation কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) registry credentials + private mirror ব্যবহার
B) pull backoff বাড়ানো
C) kubelet restart
D) node সংখ্যা কমানো

5️⃣ শেখার লক্ষ্য (Explicit)
Registry rate limit এ scaling fail বুঝে safe fix নেওয়া।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: pull error confirm
```
kubectl describe pod api-7f9d -n api | rg -n 'Failed|pull'
```

### Step 2: registry secret যোগ করো
```
kubectl -n api create secret docker-registry regcred --docker-server=docker.io --docker-username=... --docker-password=...
kubectl -n api patch serviceaccount default -p '{"imagePullSecrets":[{"name":"regcred"}]}'
```

### Step 3: retry observe
```
kubectl -n api get pods
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: authenticated pull বা mirror rate limit এড়ায়।
- কেন অন্যগুলো নয়: B/C/D root cause নয়।

### Simple Variations
- critical images pre-pull করে রাখো

## Short End-to-End (খুব সহজ)
1) Check: ImagePullBackOff + rate limit error দেখো।
2) Fix: registry secret যোগ করো বা mirror ব্যবহার করো।
3) Verify: pod pulls সফল হচ্ছে কিনা দেখো।
