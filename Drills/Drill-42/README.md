# ডেপথ ড্রিল 42: Etcd Quorum Loss from Network Partition

1️⃣ প্রোডাকশন পরিস্থিতি
3-node etcd cluster network maintenance এর পরে quorum হারিয়েছে। API server down।

2️⃣ প্রমাণ
```
$ ETCDCTL_API=3 etcdctl endpoint status -w table
+-------------------+---------+---------+-----------+
|     ENDPOINT      | VERSION | DB SIZE | IS LEADER |
+-------------------+---------+---------+-----------+
| https://10.0.0.10 |   3.5.9 |   92 GB |      true |
| https://10.0.0.11 | timeout |         |           |
| https://10.0.0.12 | timeout |         |           |

$ ping 10.0.0.11
Request timeout

$ journalctl -u etcd | tail -n 3
rafthttp: failed to reach peer
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Control-plane restore করতে safest immediate action কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) network connectivity restore করা
B) single-node cluster force করা
C) failed peers delete করা
D) kube-apiserver restart

5️⃣ শেখার লক্ষ্য (Explicit)
Etcd quorum safety বজায় রাখা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: peer reachability check
```
ping 10.0.0.11
```

### Step 2: network fix
- firewall/route ঠিক করো

### Step 3: quorum verify
```
ETCDCTL_API=3 etcdctl endpoint status -w table
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: quorum ছাড়া etcd unsafe; network restore safest।
- কেন অন্যগুলো নয়: B/C data risk, D root cause নয়।

### Simple Variations
- etcd nodes multi-AZ রাখলে partition ঝুঁকি কমে

## Short End-to-End (খুব সহজ)
1) Check: peers reachable কিনা ping করো।
2) Fix: network connectivity restore করো।
3) Verify: etcd quorum ফিরে এসেছে কিনা দেখো।
