# ডেপথ ড্রিল 09: PostgreSQL Bloat and Autovacuum Lag

1️⃣ প্রোডাকশন পরিস্থিতি
Multi-tenant Postgres (5k TPS) এ p95 latency 40ms থেকে 400ms। downtime নেওয়া যাবে না।

2️⃣ প্রমাণ
```
$ psql -c "select relname, n_dead_tup from pg_stat_user_tables order by n_dead_tup desc limit 3;"
orders  18723321
users   9021121

$ psql -c "select relname, last_autovacuum, autovacuum_count from pg_stat_user_tables where relname='orders';"
orders | 2024-05-01 02:11:10 | 3

$ psql -c "select * from pg_stat_progress_vacuum;"
(0 rows)

$ psql -c "show autovacuum_vacuum_cost_limit;"
autovacuum_vacuum_cost_limit | 200
```

3️⃣ সিদ্ধান্ত প্রশ্ন
downtime ছাড়া safest immediate action কী?

4️⃣ সম্ভাব্য পথ (3–6)
A) VACUUM FULL চালানো
B) autovacuum cost limit বাড়িয়ে targeted vacuum
C) সব read replica-তে পাঠানো
D) shared_buffers বাড়ানো

5️⃣ শেখার লক্ষ্য (Explicit)
bloat কমাতে safe vacuum strategy নেওয়া।

6️⃣ Candidate Response Requirements
- Chosen option(s)
- Commands / snippets
- অন্য পথগুলো কেন নয়

---

## সহজ Walkthrough
### Step 1: dead tuples দেখা
```
psql -c "select relname, n_dead_tup from pg_stat_user_tables order by n_dead_tup desc limit 3;"
```

### Step 2: autovacuum শক্তিশালী করো
```
psql -c "alter system set autovacuum_vacuum_cost_limit=2000;"
psql -c "select pg_reload_conf();"
```

### Step 3: targeted vacuum
```
psql -c "vacuum (analyze, verbose) orders;"
```

---

## Solution (Simple)
- Chosen option(s): B
- কেন ঠিক: VACUUM FULL locking ঝুঁকি; autovacuum tuning safe।
- কেন অন্যগুলো নয়: A downtime ঝুঁকি, C/D root cause ঠিক করে না।

### Simple Variations
- বড় টেবিল partition করলে bloat কমে

## Short End-to-End (খুব সহজ)
1) Check: dead tuples বেশি কিনা দেখো।
2) Fix: autovacuum cost limit বাড়িয়ে targeted vacuum করো।
3) Verify: latency নামছে কিনা দেখো।
