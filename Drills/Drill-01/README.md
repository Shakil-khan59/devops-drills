# ডেপথ ড্রিল 01: Etcd Latency and API Timeouts

1️⃣ প্রোডাকশন পরিস্থিতি
120-node Kubernetes cluster-এ peak deploy সময় `kubectl` timeout হচ্ছে। SLO হলো 200ms p95। 30 মিনিটের মধ্যে স্থিতিশীল করতে হবে, downtime ছাড়া।

সংক্ষিপ্ত context:
- নতুন controller 40k objects প্রতি 30s reconcile করছে।
- etcd এবং container logs একই NVMe ডিস্ক শেয়ার করছে।

2️⃣ প্রমাণ
```
$ kubectl get pods -A --request-timeout=5s
Error from server (Timeout): the server was unable to return a response in the time allotted

$ ETCDCTL_API=3 etcdctl --endpoints=https://10.0.0.10:2379 endpoint status -w table
... DB SIZE 96 GB ... IS LEADER true

$ curl -s http://127.0.0.1:2381/metrics | rg 'commit|fsync'
etcd_disk_wal_fsync_duration_seconds_sum 312.8
etcd_server_commit_duration_seconds_sum 290.1

$ iostat -x 1 3
Device   await  %util
nvme0n1  45.2   98.7
```

3️⃣ সিদ্ধান্ত প্রশ্ন
সবচেয়ে নিরাপদ immediate fix কী, আর পুনরাবৃত্তি ঠেকাতে next step কী হবে?

4️⃣ সম্ভাব্য পথ (3–6)
A) API server timeout বাড়ানো
B) এখনই etcd defrag চালানো
C) write pressure কমানো এবং disk contention কমানো
D) আরও kube-apiserver যোগ করা

5️⃣ শেখার লক্ষ্য (Explicit)
etcd disk IO bottleneck চিনে safe mitigation বেছে নেওয়া।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough (End-to-End)

### Step 1: bottleneck নিশ্চিত করো
```
iostat -x 1 5
curl -s http://127.0.0.1:2381/metrics | rg 'commit|fsync'
```

### Step 2: write pressure কমাও
```
kubectl -n <controller-namespace> scale deploy/custom-controller --replicas=0
```

### Step 3: follow-up fix
- etcd data আলাদা/দ্রুত ডিস্কে নেওয়া
- container logs আলাদা ডিস্কে নেওয়া

---

## Solution (Simple)
- Chosen option(s): C
- Commands / snippets:
```
# pause noisy controller
kubectl -n <controller-namespace> scale deploy/custom-controller --replicas=0

# verify IO + etcd latency
iostat -x 1 5
curl -s http://127.0.0.1:2381/metrics | rg 'commit|fsync'
```
- কেন এটা ঠিক: disk saturation ও etcd commit latency স্পষ্ট; write pressure কমালে safest recovery হয়।
- কেন অন্যগুলো নয়: A শুধু symptom ঢাকে, B peak-এ আরও IO চাপায়, D etcd bottleneck ঠিক করে না।

### দ্রুত যাচাই
```
iostat -x 1 5
kubectl get --raw /metrics | rg 'apiserver_request_duration_seconds_bucket' | tail -n 3
```

### Simple Variations
- etcd ঠিক, API ধীর: admission webhook চেক করো
- শুধু write ধীর: webhook timeout/CRD validation দেখো

## Short End-to-End (খুব সহজ)
1) Check: `iostat -x` আর etcd metrics (commit/fsync) দেখো।
2) Fix: noisy controller scale down করো।
3) Verify: API latency কমছে কিনা দেখো।
