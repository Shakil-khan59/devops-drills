# ডেপথ ড্রিল 28: Log Ingestion Backpressure

1️⃣ প্রোডাকশন পরিস্থিতি
Fluent Bit pipeline 2TB/day পাঠায়। spike এ logs drop হচ্ছে।

2️⃣ প্রমাণ
```
$ kubectl logs -n logging daemonset/fluent-bit | tail -n 5
[warn] [output] failed to flush chunk '1-171543' ret=retry
[error] [buffer] retry queue is full

$ kubectl get cm -n logging fluent-bit-config -o yaml | rg -n 'mem_buf_limit|storage'
Mem_Buf_Limit  50MB
storage.total_limit_size  1G

$ kubectl top pod -n logging | rg fluent-bit
CPU: 900m  Memory: 1.8Gi
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Log loss কমাতে best immediate mitigation কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) buffer limits + storage বাড়ানো
B) noisy logs বন্ধ করা
C) Fluent Bit restart করা
D) log verbosity বাড়ানো

5️⃣ শেখার লক্ষ্য (Explicit)
Backpressure এ buffer capacity tuning।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: buffer config দেখো
```
kubectl get cm -n logging fluent-bit-config -o yaml | rg -n 'mem_buf_limit|storage'
```

### Step 2: buffer বাড়াও
```
kubectl -n logging edit cm fluent-bit-config
kubectl -n logging rollout restart ds/fluent-bit
```

### Step 3: drop কমছে কিনা দেখো
```
kubectl logs -n logging daemonset/fluent-bit | tail -n 5
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: buffer সীমা কম হওয়ায় drop হচ্ছিল; capacity বাড়ালে recover হয়।
- কেন অন্যগুলো নয়: B data loss, C temporary, D load বাড়ায়।

### Simple Variations
- disk-backed buffering ব্যবহার করলে spike tolerate হয়

## Short End-to-End (খুব সহজ)
1) Check: buffer full error আছে কিনা দেখো।
2) Fix: mem/disk buffer limits বাড়াও।
3) Verify: log drop কমছে কিনা দেখো।
