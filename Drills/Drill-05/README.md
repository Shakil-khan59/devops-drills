# ডেপথ ড্রিল 05: Kafka ISR Shrink Under Disk Pressure

1️⃣ প্রোডাকশন পরিস্থিতি
Kafka cluster (12 brokers, 1.2TB/day) এ producers timeout হচ্ছে। `acks=all`। ISR shrink হয়েছে। Data loss চলবে না।

2️⃣ প্রমাণ
```
$ kafka-topics.sh --bootstrap-server b1:9092 --describe --topic orders
Partition: 3  Leader: 4  Replicas: 4,6,8  Isr: 4,6

$ kafka-log-dirs.sh --bootstrap-server b4:9092 --describe | rg -n 'ERROR|size'
logDir: /kafka-logs  totalSize: 980000000000  usable: 12000000000

$ journalctl -u kafka | tail -n 5
[WARN] Shrinking ISR from 3 to 2 for partition orders-3
[ERROR] Disk is full, shutting down
```

3️⃣ সিদ্ধান্ত প্রশ্ন
সবচেয়ে safe first move কী, যাতে durability ঠিক থাকে?

4️⃣ সম্ভাব্য পথ (3–6)
A) `min.insync.replicas=1` করা
B) disk capacity বাড়ানো বা partitions move করা
C) producer `acks=1` করা
D) পুরনো topic delete করা

5️⃣ শেখার লক্ষ্য (Explicit)
ISR shrink হলে disk pressure কীভাবে handle করতে হয়।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: disk pressure confirm
```
kafka-log-dirs.sh --bootstrap-server b4:9092 --describe
```

### Step 2: capacity বাড়াও/partition move করো
```
kafka-reassign-partitions.sh --bootstrap-server b1:9092 --generate --topics-to-move-json-file topics.json --broker-list 4,6,8
```

### Step 3: ISR back to normal দেখো
```
kafka-topics.sh --bootstrap-server b1:9092 --describe --topic orders
```

---

## Solution (Simple)
- Chosen option(s): B
- কেন ঠিক: disk full হলে ISR shrink হয়; capacity ফিরলে durability ঠিক থাকে।
- কেন অন্যগুলো নয়: A/C durability কমায়, D data loss ঝুঁকি।

### Simple Variations
- retention policy review করে proactive capacity রাখো

## Short End-to-End (খুব সহজ)
1) Check: disk full/usable bytes দেখো।
2) Fix: capacity বাড়াও বা partitions move করো।
3) Verify: ISR normal হচ্ছে কিনা দেখো।
