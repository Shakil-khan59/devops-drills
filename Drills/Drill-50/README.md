# ডেপথ ড্রিল 50: SQS Visibility Timeout Misconfig

1️⃣ প্রোডাকশন পরিস্থিতি
SQS worker 2k msgs/s। deploy পরে duplicate processing 12%। downstream overload।

2️⃣ প্রমাণ
```
$ aws sqs get-queue-attributes --queue-url https://sqs... --attribute-names VisibilityTimeout
VisibilityTimeout: 30

$ worker logs | tail -n 3
processed message id=abc123 in 55s

$ cloudwatch metrics
ApproximateNumberOfMessagesNotVisible spikes during peak
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Duplicate কমাতে best fix কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) visibility timeout processing time-এর উপরে সেট + heartbeat
B) timeout কমানো
C) retries বন্ধ করা
D) batch processing চালু করা

5️⃣ শেখার লক্ষ্য (Explicit)
SQS visibility timeout সঠিকভাবে align করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: processing time confirm
```
# logs দেখে max processing time বের করো
```

### Step 2: visibility timeout বাড়াও
```
aws sqs set-queue-attributes --queue-url https://sqs... --attributes VisibilityTimeout=120
```

### Step 3: heartbeat ব্যবহার
```
# ChangeMessageVisibility during long jobs
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: processing time > visibility timeout হলে duplicate হয়; timeout বাড়ালে ঠিক হয়।
- কেন অন্যগুলো নয়: B problem বাড়ায়, C reliability কমায়, D root cause নয়।

### Simple Variations
- idempotent handlers যোগ করলে duplicate safe হয়

## Short End-to-End (খুব সহজ)
1) Check: processing time > visibility timeout কিনা দেখো।
2) Fix: visibility timeout বাড়াও + heartbeat দাও।
3) Verify: duplicates কমছে কিনা দেখো।
