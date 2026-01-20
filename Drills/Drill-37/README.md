# ডেপথ ড্রিল 37: TLS Chain Break After Intermediate Expiry

1️⃣ প্রোডাকশন পরিস্থিতি
Cert renewal এর পরে কিছু region-এ TLS error। global traffic 20% কমেছে।

2️⃣ প্রমাণ
```
$ echo | openssl s_client -connect api.example.com:443 -servername api.example.com 2>/dev/null | rg -n 'Verify return|issuer'
Verify return code: 21 (unable to verify the first certificate)
issuer=/C=US/O=Example/CN=Example Intermediate 2022

$ openssl x509 -in intermediate.crt -noout -enddate
notAfter=May 10 23:59:59 2024 GMT

$ curl -v https://api.example.com
SSL certificate problem: unable to get local issuer certificate
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Trust restore করতে correct fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) নতুন intermediate + full chain serve করা
B) HTTP এ নামা
C) client verification বন্ধ করা
D) self-signed cert ব্যবহার

5️⃣ শেখার লক্ষ্য (Explicit)
TLS chain সঠিকভাবে serve করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: chain verify
```
openssl s_client -connect api.example.com:443 -servername api.example.com
```

### Step 2: fullchain install
- server + new intermediate + root chain আপডেট করো

### Step 3: verify again
```
curl -v https://api.example.com
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: intermediate expire হলে chain incomplete হয়; full chain দিলে trust ফেরে।
- কেন অন্যগুলো নয়: B/C/D security ভাঙে।

### Simple Variations
- CI তে cert chain validation যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: openssl s_client এ verify error আছে কিনা দেখো।
2) Fix: full chain (server+intermediate) install করো।
3) Verify: curl TLS error নেই কিনা দেখো।
