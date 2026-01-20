# ডেপথ ড্রিল 06: Terraform State Lock Deadlock

1️⃣ প্রোডাকশন পরিস্থিতি
Terraform CI pipeline 30 মিনিট ধরে state lock error দিচ্ছে। Hotfix deploy আটকে আছে।

2️⃣ প্রমাণ
```
$ terraform apply
Error: Error acquiring the state lock
Lock Info:
  ID:        1a2b3c4d
  Path:      prod/network/terraform.tfstate
  Operation: OperationTypeApply
  Who:       runner@ci-42
  Version:   1.5.2

$ aws dynamodb get-item --table-name tf-locks --key '{"LockID":{"S":"prod/network/terraform.tfstate"}}'
"Info": {"S": "{\"ID\":\"1a2b3c4d\",\"Operation\":\"Apply\",\"Created\":\"2024-05-11T14:02:19Z\"}"}

$ aws s3 ls s3://tf-state/prod/network/terraform.tfstate
2024-05-11 13:58:04  18472 terraform.tfstate
```

3️⃣ সিদ্ধান্ত প্রশ্ন
state corruption না করে safe unlock কীভাবে করবে?

4️⃣ সম্ভাব্য পথ (3–6)
A) যাচাই ছাড়াই force-unlock
B) runner dead কিনা যাচাই করে force-unlock
C) S3 state ম্যানুয়ালি edit করা
D) state locking বন্ধ করা

5️⃣ শেখার লক্ষ্য (Explicit)
Terraform lock contention নিরাপদে সমাধান করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: lock metadata দেখো
```
aws dynamodb get-item --table-name tf-locks --key '{"LockID":{"S":"prod/network/terraform.tfstate"}}'
```

### Step 2: runner সত্যিই dead কিনা নিশ্চিত করো
- CI job status চেক

### Step 3: তারপর force-unlock
```
terraform force-unlock 1a2b3c4d
```

---

## Solution (Simple)
- Chosen option(s): B
- কেন ঠিক: যাচাই ছাড়া unlock করলে concurrent apply হতে পারে।
- কেন অন্যগুলো নয়: A ঝুঁকিপূর্ণ, C/D unsafe।

### Simple Variations
- stale lock cleanup automation যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: lock metadata আর CI job status দেখো।
2) Fix: dead হলে `terraform force-unlock`।
3) Verify: apply আবার রান হয়ে কিনা দেখো।
