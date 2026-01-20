# ডেপথ ড্রিল 13: EBS IO Credit Exhaustion

1️⃣ প্রোডাকশন পরিস্থিতি
PostgreSQL primary gp2 EBS এ চলছে। reporting job এর পরে p99 latency বেড়ে গেছে। Writes বন্ধ করা যাবে না।

2️⃣ প্রমাণ
```
$ aws cloudwatch get-metric-statistics --metric-name BurstBalance --namespace AWS/EBS --dimensions Name=VolumeId,Value=vol-abc
Datapoints: [ {"Average": 0.0, "Timestamp": "2024-05-11T12:04:00Z"} ]

$ iostat -x 1 3
Device            r/s   w/s  await  %util
nvme1n1          220  6800   56.8   99.2

$ psql -c "select now()-query_start, state, query from pg_stat_activity where state='active' limit 3;"
00:00:12 | active | INSERT INTO events ...
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Durability রেখে immediate mitigation কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) gp3 তে migrate করে IOPS বাড়ানো
B) synchronous_commit বন্ধ করা
C) shared_buffers বাড়ানো
D) DB restart

5️⃣ শেখার লক্ষ্য (Explicit)
EBS performance limit চিনে safe storage fix নেওয়া।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: BurstBalance দেখো
```
aws cloudwatch get-metric-statistics --metric-name BurstBalance --namespace AWS/EBS --dimensions Name=VolumeId,Value=vol-abc
```

### Step 2: volume modify
```
aws ec2 modify-volume --volume-id vol-abc --volume-type gp3 --iops 12000 --throughput 500
```

### Step 3: latency normal হচ্ছে কিনা দেখো
```
iostat -x 1 5
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: gp2 burst credit শেষ; gp3 দিয়ে guaranteed IOPS পাওয়া যায়।
- কেন অন্যগুলো নয়: B durability কমায়, C root cause নয়, D risky।

### Simple Variations
- workload bursty হলে baseline IOPS আগে থেকেই রাখো

## Short End-to-End (খুব সহজ)
1) Check: EBS BurstBalance শূন্য কিনা দেখো।
2) Fix: gp3 তে migrate করে IOPS বাড়াও।
3) Verify: iostat latency কমছে কিনা দেখো।
