# ডেপথ ড্রিল 49: NTP Drift Breaks JWT Validation

1️⃣ প্রোডাকশন পরিস্থিতি
300 nodes-এ JWT validation intermittent fail। error spike হয়। auth critical।

2️⃣ প্রমাণ
```
$ app logs | tail -n 3
JWT invalid: token used before issued

$ chronyc tracking
Last offset     : -2.340 seconds
RMS offset      : 1.120 seconds

$ date -u
Sat May 11 12:12:31 UTC 2024
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Auth stabilize করতে correct fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) NTP drift ঠিক করা
B) JWT leeway 10 মিনিট করা
C) `iat` validation বন্ধ করা
D) auth services restart

5️⃣ শেখার লক্ষ্য (Explicit)
Time sync সমস্যা security token ভাঙে বোঝা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: drift চেক
```
chronyc tracking
```

### Step 2: time sync fix
```
chronyc makestep
```

### Step 3: JWT errors monitor
- auth logs পর্যবেক্ষণ

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: clock skew থাকলে iat/exp check fail হয়।
- কেন অন্যগুলো নয়: B/C security কমায়, D root cause নয়।

### Simple Variations
- NTP alerting সেট করো

## Short End-to-End (খুব সহজ)
1) Check: NTP drift (`chronyc tracking`) দেখো।
2) Fix: time sync করো।
3) Verify: JWT errors কমছে কিনা দেখো।
