# ডেপথ ড্রিল 29: DNS TTL Blocks Fast Failover

1️⃣ প্রোডাকশন পরিস্থিতি
Primary region down হলে DNS failover হলেও clients 20 মিনিট dead region-এ যাচ্ছে। SLA 60s failover।

2️⃣ প্রমাণ
```
$ dig +noall +answer api.example.com
api.example.com.  1800  IN  A  54.12.34.56

$ aws route53 list-resource-record-sets --hosted-zone-id Z123 | rg api.example.com -A 2
TTL: 1800

$ aws route53 get-health-check-status --health-check-id 456
Status: HealthCheckFailure
```

3️⃣ সিদ্ধান্ত প্রশ্ন
60s failover করতে best fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) TTL 30s করা + health checks ঠিক রাখা
B) TTL বাড়ানো
C) health checks বন্ধ করা
D) latency routing চালু রাখা (failover ছাড়া)

5️⃣ শেখার লক্ষ্য (Explicit)
DNS TTL failover time কিভাবে control করে।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: বর্তমান TTL দেখো
```
aws route53 list-resource-record-sets --hosted-zone-id Z123
```

### Step 2: TTL কমাও
```
aws route53 change-resource-record-sets --hosted-zone-id Z123 --change-batch file://ttl-30s.json
```

### Step 3: failover test
- health check fail হলে 60s এর মধ্যে route change হচ্ছে কিনা দেখো

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: TTL বেশি হলে cached IP দেরিতে বদলায়।
- কেন অন্যগুলো নয়: B failover ধীর, C automation হারায়, D SLA মিস।

### Simple Variations
- critical endpoint এ কম TTL, others এ বেশি TTL

## Short End-to-End (খুব সহজ)
1) Check: DNS TTL কত তা দেখো।
2) Fix: TTL 30–60s এ নামাও।
3) Verify: failover দ্রুত হচ্ছে কিনা দেখো।
