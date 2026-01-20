# ডেপথ ড্রিল 27: Elasticsearch Yellow Cluster

1️⃣ প্রোডাকশন পরিস্থিতি
20-node Elasticsearch (6TB) yellow state, search latency বাড়ছে। data loss চলবে না।

2️⃣ প্রমাণ
```
$ curl -s localhost:9200/_cluster/health?pretty
"status" : "yellow",
"unassigned_shards" : 24

$ curl -s localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason | head -n 3
logs-2024.05  3 r UNASSIGNED NODE_LEFT

$ curl -s localhost:9200/_nodes/stats/fs | rg -n 'available_in_bytes'
"available_in_bytes" : 124121088
```

3️⃣ সিদ্ধান্ত প্রশ্ন
Data loss ছাড়াই health ফিরাতে best step কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) storage/nodes বাড়িয়ে shard reroute
B) empty primary force-allocate
C) replica count বাড়ানো
D) indices delete

5️⃣ শেখার লক্ষ্য (Explicit)
Disk pressure + shard unassignment handle করা।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: allocation explain
```
curl -s localhost:9200/_cluster/allocation/explain?pretty
```

### Step 2: capacity বাড়াও
- নতুন node বা disk যোগ করো

### Step 3: reroute
```
curl -s -XPOST localhost:9200/_cluster/reroute?pretty
```

---

## Solution (Simple)
- Chosen option(s): A
- কেন ঠিক: capacity না বাড়ালে replicas assign হয় না।
- কেন অন্যগুলো নয়: B data loss, C unassigned ঠিক করে না, D risky।

### Simple Variations
- ILM দিয়ে hot/warm tier করো

## Short End-to-End (খুব সহজ)
1) Check: unassigned shards এবং disk free দেখো।
2) Fix: capacity বাড়িয়ে shard reroute করো।
3) Verify: cluster status green/yellow ভালো হচ্ছে কিনা দেখো।
