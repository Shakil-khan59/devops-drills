# ডেপথ ড্রিল 26: MySQL Replication Lag

1️⃣ প্রোডাকশন পরিস্থিতি
MySQL primary 4k TPS, replicas lag 15 মিনিট। dashboards ভেঙে যাচ্ছে।

2️⃣ প্রমাণ
```
mysql> SHOW SLAVE STATUS\G
Seconds_Behind_Master: 920
Slave_SQL_Running: Yes
Relay_Log_Space: 1840000000

mysql> SHOW PROCESSLIST;
Id 42  State: Updating  Time: 712  Info: UPDATE orders SET ...

mysql> SHOW ENGINE INNODB STATUS\G
history list length 38219
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Lag কমাতে first safe action কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) long-running transaction kill করা (safe হলে)
B) lagging replica promote করা
C) binary logging বন্ধ করা
D) max_connections বাড়ানো

5️⃣ শেখার লক্ষ্য (Explicit)
Long transaction replication lag বাড়ায় বোঝা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: long query খুঁজো
```
SHOW PROCESSLIST;
```

### Step 2: safe হলে kill
```
KILL 42;
```

### Step 3: lag monitor
```
SHOW SLAVE STATUS\G
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: দীর্ঘ transaction replica apply আটকে দেয়।
- কেন অন্যগুলো নয়: B stale data risk, C replication ভাঙে, D unrelated।

### Simple Variations
- query timeout + index tuning করলে ভবিষ্যৎ lag কমে

## Short End-to-End (খুব সহজ)
1) Check: long-running query আছে কিনা দেখো।
2) Fix: safe হলে query kill করো।
3) Verify: Seconds_Behind_Master কমছে কিনা দেখো।
