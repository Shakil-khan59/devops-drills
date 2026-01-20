# ডেপথ ড্রিল 16: Vault Token Renewal Failing

1️⃣ প্রোডাকশন পরিস্থিতি
Vault periodic token ব্যবহার করা অ্যাপগুলোতে auth failure হচ্ছে। tokens দ্রুত expire হচ্ছে।

2️⃣ প্রমাণ
```
$ journalctl -u vault | tail -n 5
error renewing token: permission denied
error renewing token: context deadline exceeded

$ vault token lookup
period: 1h
ttl: 4m

$ chronyc tracking
Last offset     : +1.782 seconds
Root dispersion : 3.020 seconds
```

3️⃣ সিদ্ধান্ত প্রশ্ন
সবচেয়ে সম্ভাব্য root cause এবং fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) NTP time drift ঠিক করা
B) TTL 24h করা
C) token renewal বন্ধ করা
D) Vault restart

5️⃣ শেখার লক্ষ্য (Explicit)
clock skew কীভাবে token renewal ভাঙে তা বোঝা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: time drift চেক
```
chronyc tracking
```

### Step 2: time sync ঠিক করো
```
chronyc makestep
```

### Step 3: token renew test
```
vault token renew
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: clock skew থাকলে token TTL ভুল হয়।
- কেন অন্যগুলো নয়: B/C security issue, D root cause নয়।

### Simple Variations
- NTP monitoring alert যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: `chronyc tracking` এ drift আছে কিনা দেখো।
2) Fix: NTP sync করো।
3) Verify: Vault token renew কাজ করছে কিনা দেখো।
