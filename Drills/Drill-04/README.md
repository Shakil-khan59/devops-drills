# ডেপথ ড্রিল 04: Redis Cluster Failover Latency

1️⃣ প্রোডাকশন পরিস্থিতি
12-node Redis Cluster (40k ops/s) এ failover-এর পরে p99 latency 400ms। SLO 50ms। session state preserve করতে হবে।

2️⃣ প্রমাণ
```
$ redis-cli -c cluster nodes | head -n 3
07c3... 10.0.3.7:6379 master,fail?
9ab1... 10.0.3.8:6379 slave 07c3...

$ redis-cli -c info stats | rg -n 'instantaneous_ops|sync'
instantaneous_ops_per_sec:10500
sync_partial_err:112

$ redis-cli -c slowlog get 3
1) 1) (integer) 112
   2) (integer) 170221
   3) (integer) 650
   4) 1) "EVALSHA" ...

$ redis-cli -c info memory | rg -n 'used_memory_human|mem_fragmentation_ratio'
used_memory_human:29.3G
mem_fragmentation_ratio:1.89
```

3️⃣ সিদ্ধান্ত প্রশ্ন
কোন immediate action latency কমাবে এবং consistency বজায় রাখবে?

4️⃣ সম্ভাব্য পথ (3–6)
A) পুরনো master-এ fail back করা
B) client timeout বাড়ানো
C) cluster rebalance + hot-key scripts কমানো
D) AOF বন্ধ করা

5️⃣ শেখার লক্ষ্য (Explicit)
failover পরবর্তী latency কারণ বুঝে safe mitigation নেওয়া।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: slow commands খুঁজো
```
redis-cli -c slowlog get 10
```

### Step 2: hot slots rebalance
```
redis-cli -c --cluster rebalance 10.0.3.7:6379
```

### Step 3: script-heavy workload কমাও
- heavy EVALSHA কমানো বা cache করা

---

## Solution (Simple)
- Chosen option(s): C
- কেন ঠিক: failover + hot keys মিলেই latency; rebalance + script tuning safer।
- কেন অন্যগুলো নয়: A/B symptom ঢাকে, D durability কমায়।

### Simple Variations
- memory fragmentation বেশি হলে restart window পরিকল্পনা করো

## Short End-to-End (খুব সহজ)
1) Check: `slowlog` এ heavy command আছে কিনা দেখো।
2) Fix: cluster rebalance + hot keys কমাও।
3) Verify: p99 latency নামছে কিনা দেখো।
