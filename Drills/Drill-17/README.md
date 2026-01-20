# ডেপথ ড্রিল 17: Prometheus WAL Corruption

1️⃣ প্রোডাকশন পরিস্থিতি
Prometheus প্রতি 10 মিনিটে restart হচ্ছে। Alerts miss হচ্ছে।

2️⃣ প্রমাণ
```
$ kubectl logs -n monitoring prometheus-0 | tail -n 6
level=error ts=... msg="opening storage failed" err="open /prometheus/wal/00001234: invalid checksum"
level=info ts=... msg="Starting TSDB"

$ promtool tsdb analyze /prometheus | tail -n 3
block: 01HW... corrupt: true

$ kubectl get pvc -n monitoring prometheus-db-prometheus-0
STATUS: Bound  STORAGECLASS: gp2
```

3️⃣ সিদ্ধান্ত প্রশ্ন
মিনিমাল data loss রেখে recovery কীভাবে করবে?

4️⃣ সম্ভাব্য পথ (3–6)
A) WAL delete করে Prometheus চালু করা
B) replicas বাড়ানো
C) সপ্তাহ-পুরনো snapshot restore
D) WAL disable করা

5️⃣ শেখার লক্ষ্য (Explicit)
TSDB corruption recover করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: corruption confirm
```
promtool tsdb analyze /prometheus
```

### Step 2: WAL remove
```
rm -rf /prometheus/wal
```

### Step 3: pod restart
```
kubectl -n monitoring delete pod prometheus-0
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: WAL corrupt হলে start হয় না; remove করলে দ্রুত recover হয় (recent data loss accept)।
- কেন অন্যগুলো নয়: B root cause নয়, C data loss বেশি, D unsafe।

### Simple Variations
- HA Prometheus বা remote_write থাকলে loss কমে

## Short End-to-End (খুব সহজ)
1) Check: WAL corruption error আছে কিনা দেখো।
2) Fix: WAL delete করে pod restart করো।
3) Verify: Prometheus up এবং scrape ok কিনা দেখো।
