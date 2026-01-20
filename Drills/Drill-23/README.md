# ডেপথ ড্রিল 23: JVM GC Pause Spikes

1️⃣ প্রোডাকশন পরিস্থিতি
Java service (3k rps) এ p99 latency 1.8s। নতুন feature পরে GC pause বেড়েছে। rollback এড়াতে হবে।

2️⃣ প্রমাণ
```
$ jstat -gcutil 12345 1s 3
S0 S1 E O M CCS YGC YGCT FGC FGCT GCT
0.0 0.0 85 92 95 90 402 7.12 14 28.9 36.0

$ grep "Pause Full" /var/log/app/gc.log | tail -n 2
Pause Full (G1 Evacuation Pause) 1800ms

$ kubectl top pod api-7f9d
CPU: 1.8 cores  Memory: 3.6Gi
```

3️⃣ সিদ্ধান্ত প্রশ্ন
p99 কমাতে safest mitigation কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) heap বাড়ানো + G1 GC tune করা
B) GC logs বন্ধ করা
C) client retry বাড়ানো
D) CPU limit কমানো

5️⃣ শেখার লক্ষ্য (Explicit)
GC pause এবং latency সম্পর্ক বোঝা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: GC pause confirm
```
grep "Pause Full" /var/log/app/gc.log | tail -n 5
```

### Step 2: heap + GC tune
```
kubectl -n api set env deploy/api JAVA_TOOL_OPTIONS="-Xms2g -Xmx2g -XX:+UseG1GC -XX:MaxGCPauseMillis=200"
```

### Step 3: latency observe
```
kubectl -n api top pod
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: full GC কমলে p99 কমে।
- কেন অন্যগুলো নয়: B/C symptom ঢাকে, D worsen করে।

### Simple Variations
- allocation hotspot profiling করলে root cause কমে

## Short End-to-End (খুব সহজ)
1) Check: GC logs-এ Full GC আছে কিনা দেখো।
2) Fix: heap size বাড়িয়ে G1 tuning করো।
3) Verify: p99 latency কমছে কিনা দেখো।
