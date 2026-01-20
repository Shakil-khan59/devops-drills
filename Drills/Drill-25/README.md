# ডেপথ ড্রিল 25: RabbitMQ Queue Backlog

1️⃣ প্রোডাকশন পরিস্থিতি
Queue 15 মিলিয়ন messages এ উঠেছে। Consumers আছে কিন্তু throughput flat। SLA 5 মিনিট।

2️⃣ প্রমাণ
```
$ rabbitmqctl list_queues name messages_ready messages_unacknowledged
orders 15000000 800000

$ rabbitmqctl list_consumers | head -n 3
pid: <0.123>  prefetch: 1000  ack_required: true

$ rabbitmqctl list_connections name state channels
conn-1 running 120
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Backlog কমাতে safest action কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) prefetch কমিয়ে consumer concurrency বাড়ানো
B) queue mirroring বাড়ানো
C) acks বন্ধ করা
D) queue purge

5️⃣ শেখার লক্ষ্য (Explicit)
Flow control ও consumer tuning বোঝা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: prefetch দেখতে হবে
```
rabbitmqctl list_consumers | head -n 5
```

### Step 2: prefetch কমাও + consumer scale
```
# app config এ prefetch কমাও
kubectl -n workers scale deploy/orders-consumer --replicas=50
```

### Step 3: backlog কমছে কিনা দেখো
```
rabbitmqctl list_queues name messages_ready messages_unacknowledged
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: বেশি prefetch ack delay বাড়ায়; concurrency বাড়ালে throughput বাড়ে।
- কেন অন্যগুলো নয়: B overhead, C correctness ভাঙে, D data loss।

### Simple Variations
- consumer autoscaling policy যোগ করো

## Short End-to-End (খুব সহজ)
1) Check: prefetch খুব বেশি কিনা দেখো।
2) Fix: prefetch কমাও + consumers scale করো।
3) Verify: backlog কমছে কিনা দেখো।
