### Exercise 1: Index Performance

Create a table and test index performance:

```sql
-- Create table
CREATE TABLE test_users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    age INTEGER,
    status VARCHAR(20),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Insert 100k rows
INSERT INTO test_users (name, email, age, status)
SELECT 
    'User ' || generate_series,
    'user' || generate_series || '@example.com',
    (random() * 60 + 18)::INTEGER,
    CASE WHEN random() < 0.7 THEN 'active' ELSE 'inactive' END
FROM generate_series(1, 100000);

-- Query 1: No index
EXPLAIN ANALYZE SELECT * FROM test_users WHERE age > 30;

-- Create index
CREATE INDEX idx_age ON test_users(age);

-- Query 2: With index
EXPLAIN ANALYZE SELECT * FROM test_users WHERE age > 30;

-- Compare cost and actual time
```

**Questions to answer:**
1. What was the cost difference?
2. What was the actual time difference?
3. Did Postgres use the index? Why or why not?

---

**Answers**
1. There was no cost difference, as the index was not used when scanning.
2. There was minimal time difference, which varied most times, as the index was not used.
3. Postgres didn't use the index! It didn't use it because, the filter age > 30 is of very low selectivity, in which case a sequential reading of the whole table over reading rows randomly from various pages based on pointers, leads to lower cost. However, if we were to use something of high selectivity, such as age > 75, we would use the index.


**ROUGH WORK**:
-- PRE INDEX
explain ANALYZE select * from TEST_USERS
where age >30;
-- Seq Scan on test_users  (cost=0.00..2380.00 rows=79813 width=55) (actual time=0.025..32.221 rows=79339 loops=1)
-- Planning Time: 0.615 ms
-- Execution Time: 36.354 ms
--   Rows Removed by Filter: 20661

create index idx_test_users_age on test_users(age);

-- POST INDEX
explain ANALYZE select * from TEST_USERS
where age >30;
-- Seq Scan on test_users  (cost=0.00..2380.00 rows=79813 width=55) (actual time=0.014..17.736 rows=79339 loops=1)
-- Planning Time: 0.775 ms
-- Execution Time: 21.558 ms

-- FORCING INDEX USAGE
SET enable_seqscan = off;
EXPLAIN ANALYZE SELECT * FROM test_users WHERE age > 30;
SET enable_seqscan = on;
-- Bitmap Heap Scan on test_users  (cost=906.84..3034.51 rows=79813 width=55) (actual time=11.267..63.038 rows=79339 loops=1)
--  Recheck Cond: (age > 30)
--  Heap Blocks: exact=1130
--  ->  Bitmap Index Scan on idx_test_users_age  (cost=0.00..886.89 rows=79813 width=0) (actual time=11.069..11.069 rows=79339 loops=1)
--        Index Cond: (age > 30)
-- Planning Time: 0.157 ms
-- Execution Time: 66.799 ms


select * from pg_indexes;

EXPLAIN ANALYZE SELECT * FROM test_users WHERE age > 75;
-- Bitmap Heap Scan on test_users  (cost=50.39..1235.39 rows=4400 width=55) (actual time=50.364..234.759 rows=4330 loops=1)
--  Recheck Cond: (age > 75)
--  Heap Blocks: exact=1114
--  ->  Bitmap Index Scan on idx_test_users_age  (cost=0.00..49.29 rows=4400 width=0) (actual time=47.960..47.961 rows=4330 loops=1)
--        Index Cond: (age > 75)
-- Planning Time: 6.311 ms
-- Execution Time: 255.770 ms


