# ডেপথ ড্রিল 12: S3 403 with KMS Encryption

1️⃣ প্রোডাকশন পরিস্থিতি
Batch job encrypted S3 (SSE-KMS) থেকে পড়তে পারছে না। IAM change এর পরে 30TB backlog হচ্ছে।

2️⃣ প্রমাণ
```
$ aws s3 cp s3://prod-bucket/export/2024-05-11.csv -
An error occurred (AccessDenied) when calling the GetObject operation: Access Denied

$ aws kms decrypt --key-id arn:aws:kms:... --ciphertext-blob fileb://blob
An error occurred (AccessDeniedException)

$ aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=Decrypt | tail -n 2
ErrorCode: AccessDenied
Username: role/batch-reader

$ aws iam get-role-policy --role-name batch-reader --policy-name s3-read
"kms:Decrypt" not present
```

3️⃣ সিদ্ধান্ত প্রশ্ন
least privilege রেখে access restore কীভাবে করবে?

4️⃣ সম্ভাব্য পথ (3–6)
A) KMS key + bucket prefix scoped `kms:Decrypt` যোগ করা
B) KMS বন্ধ করা
C) full KMS permissions দেওয়া
D) unencrypted নতুন bucket এ কপি করা

5️⃣ শেখার লক্ষ্য (Explicit)
S3 + KMS permission debugging এবং safe fix করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: IAM policy audit
```
aws iam get-role-policy --role-name batch-reader --policy-name s3-read
```

### Step 2: missing permission যোগ করা
```
aws iam put-role-policy --role-name batch-reader --policy-name s3-read --policy-document file://kms-policy.json
```

### Step 3: read test
```
aws s3 cp s3://prod-bucket/export/2024-05-11.csv -
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: KMS decrypt missing ছিল; scoped permission দিলে secure ভাবে কাজ হয়।
- কেন অন্যগুলো নয়: B/C/D security risk।

### Simple Variations
- policy test CI তে যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: IAM policy তে `kms:Decrypt` আছে কিনা দেখো।
2) Fix: scoped permission যোগ করো।
3) Verify: S3 read কাজ করছে কিনা দেখো।
