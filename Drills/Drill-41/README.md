# ডেপথ ড্রিল 41: RDS Multi-AZ Failover Latency

1️⃣ প্রোডাকশন পরিস্থিতি
RDS Multi-AZ PostgreSQL failover এ 60s errors আর 5 মিনিট latency spike হচ্ছে।

2️⃣ প্রমাণ
```
$ aws rds describe-events --source-identifier db-prod | tail -n 2
DB instance failed over
DB instance restarted

$ app logs | tail -n 3
could not connect to server: Connection timed out

$ dig +short db-prod.cluster-xyz.us-east-1.rds.amazonaws.com
10.12.7.44
```

3️⃣ সিদ্ধান্ত প্রশ্ন
পরেরবার failover impact কমাতে best action কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) connection retry + backoff যোগ করা এবং DNS TTL কমানো
B) failover বন্ধ করা
C) instance size শুধু বাড়ানো
D) পুরনো IP pin করা

5️⃣ শেখার লক্ষ্য (Explicit)
Failover survive করার জন্য app connectivity strategy।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: failover logs confirm
```
aws rds describe-events --source-identifier db-prod | tail -n 5
```

### Step 2: app retry/backoff যোগ
- exponential backoff + short connect timeout

### Step 3: DNS cache TTL কমাও
- resolver বা app layer TTL tuning

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: failover-এ DNS/IP change হলে retry/backoff দ্রুত recover করে।
- কেন অন্যগুলো নয়: B HA ভাঙে, C guaranteed fix নয়, D brittle।

### Simple Variations
- RDS Proxy ব্যবহার করলে failover smoother হয়

## Short End-to-End (খুব সহজ)
1) Check: failover event আর app timeout দেখো।
2) Fix: retry/backoff + DNS TTL কমাও।
3) Verify: পরের failover-এ error time কমছে কিনা দেখো।
