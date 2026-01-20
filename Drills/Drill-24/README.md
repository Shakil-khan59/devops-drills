# ডেপথ ড্রিল 24: NATS Cluster Split Brain

1️⃣ প্রোডাকশন পরিস্থিতি
5-node NATS cluster এ network issue এর পরে duplicate messages হচ্ছে।

2️⃣ প্রমাণ
```
$ nats-server -sl
server id: A2C...  cluster: connected=2 routes=1
server id: B7D...  cluster: connected=2 routes=1

$ nats top
Subscriptions: 4200  InMsgs: 180k  OutMsgs: 350k

$ journalctl -u nats | tail -n 5
[ERR] Router connection closed: Read Error
[WRN] Cluster not fully connected
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Split brain দ্রুত ঠিক করার safest action কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) network partition ঠিক করা ও routes verify করা
B) সব node একসাথে restart
C) duplicate detection বন্ধ করা
D) max payload বাড়ানো

5️⃣ শেখার লক্ষ্য (Explicit)
Messaging split brain diagnose ও recover করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: cluster routes দেখো
```
nats-server -sl
```

### Step 2: network check
```
nc -vz <peer-ip> 6222
```

### Step 3: full mesh confirm
- সব nodes connected কিনা verify করো

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: partition থাকলে duplicates হয়; network fix হলেই consistency ফেরে।
- কেন অন্যগুলো নয়: B risky, C correctness ভাঙে, D irrelevant।

### Simple Variations
- multi-AZ routing health check যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: NATS cluster full connected কিনা দেখো।
2) Fix: network partition ঠিক করো।
3) Verify: duplicate messages কমছে কিনা দেখো।
