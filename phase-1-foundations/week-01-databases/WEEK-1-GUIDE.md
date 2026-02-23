# WEEK 1: HOW DATABASES ACTUALLY WORK
## Complete Learning Package

**Deadline: March 2nd, 2025 (Sunday)**

---

## 📅 DAILY SCHEDULE

### Monday (Feb 23) - Today!
- ✅ Read Concept 1: How PostgreSQL Stores Data (above)
- ✅ Read Concept 2: Query Execution Internals (below)
- ✅ Watch: Hussein Nasser - "How PostgreSQL Works" (YouTube)

### Tuesday (Feb 24)
- Read Concept 3: Indexes Deep Dive
- Read Concept 4: MVCC and Transactions
- Complete Exercise 1 & 2

### Wednesday (Feb 25)
- Read Concept 5: EXPLAIN ANALYZE Mastery
- Complete Exercise 3 & 4
- Take the quiz

### Thursday (Feb 26)
- Start project: Database Performance Detective
- Set up the slow database
- Analyze first 2 slow queries

### Friday (Feb 27)
- Continue project
- Analyze remaining queries
- Implement optimizations

### Saturday (Feb 28)
- Finish project
- Write optimization report
- Document findings

### Sunday (March 1-2)
- Polish project README
- Review interview prep questions
- Start blog post draft
- Submit everything by EOD March 2nd

---

## 📖 CONCEPT 2: QUERY EXECUTION INTERNALS

### What Happens When You Run a Query?

```sql
SELECT name, email 
FROM users 
WHERE age > 25 
ORDER BY created_at 
LIMIT 10;
```

**The Journey:**

#### 1. PARSER (Syntax Check)
- Checks if SQL is valid
- Builds a parse tree
- Catches syntax errors

#### 2. REWRITER (Apply Rules)
- Applies views (if any)
- Applies row-level security
- Rewrites query if needed

#### 3. PLANNER/OPTIMIZER (The Brain)
**This is where the magic happens!**

The planner:
- Looks at available indexes
- Estimates number of rows to scan
- Estimates cost of different approaches
- Chooses the cheapest plan

**Example Plans:**

**Option 1: Sequential Scan**
```
Cost: Read all pages → Filter → Sort → Limit
Fast if: Table is small OR need most rows
```

**Option 2: Index Scan**
```
Cost: Read index → Fetch matching rows → Sort → Limit
Fast if: Filtering eliminates most rows
```

**Option 3: Index-Only Scan**
```
Cost: Read index only (if it covers all needed columns)
Fastest if: Index contains all columns in SELECT
```

**The planner picks based on:**
- Table statistics (ANALYZE)
- Index availability
- Estimated selectivity of WHERE clause
- Available memory (work_mem)

#### 4. EXECUTOR (Does the Work)
- Executes the chosen plan
- Reads pages from disk/cache
- Applies filters
- Sorts results
- Returns rows

---

### Understanding Query Plans

**The Most Important Nodes:**

**Sequential Scan:**
```
Seq Scan on users  (cost=0.00..180.00 rows=5000 width=64)
  Filter: (age > 25)
```
- Reads every row in the table
- Applies filter
- cost=startup..total
- rows=estimated output
- width=average row size in bytes

**Index Scan:**
```
Index Scan using users_age_idx on users  
  (cost=0.42..83.52 rows=1500 width=64)
  Index Cond: (age > 25)
```
- Uses index to find matching rows
- Faster startup (0.42 vs 0.00)
- Lower total cost (83.52 vs 180.00)

**Bitmap Heap Scan:**
```
Bitmap Heap Scan on users  (cost=41.18..245.37 rows=1500 width=64)
  Recheck Cond: (age > 25)
  -> Bitmap Index Scan on users_age_idx  
       (cost=0.00..40.80 rows=1500 width=0)
       Index Cond: (age > 25)
```
- Build bitmap of matching pages
- Then read those pages
- Good for medium selectivity

**Sort:**
```
Sort  (cost=315.39..319.14 rows=1500 width=64)
  Sort Key: created_at
  -> Seq Scan on users  (cost=0.00..180.00 rows=1500 width=64)
```
- Sorts the result
- Cost depends on number of rows and available work_mem

**Limit:**
```
Limit  (cost=0.42..1.67 rows=10 width=64)
  -> Index Scan using users_created_at_idx on users  
       (cost=0.42..187.54 rows=1500 width=64)
```
- Stops after N rows
- Can dramatically reduce actual execution time

---

### Cost Estimation

**What is "cost"?**
- NOT actual time
- Abstract unit representing I/O + CPU work
- Used to compare plans

**Typical costs:**
- Sequential page read: 1.0
- Random page read: 4.0
- CPU operation: 0.01

**Example:**
```
cost=0.00..180.00
```
- Startup cost: 0.00 (no setup needed)
- Total cost: 180.00 (reading ~180 pages sequentially)

---

### Statistics and ANALYZE

**The Problem:** Planner needs to estimate costs accurately.

**The Solution:** Table statistics.

**What ANALYZE Does:**
```sql
ANALYZE users;
```
- Samples rows from the table
- Calculates statistics:
  - Number of distinct values per column
  - Most common values
  - Distribution of values (histogram)
  - Correlation with physical storage order

**Check statistics:**
```sql
SELECT * FROM pg_stats WHERE tablename = 'users';
```

**Why This Matters:**
- Outdated stats → bad query plans
- After bulk INSERT/UPDATE → run ANALYZE
- Large tables: ANALYZE runs automatically (autovacuum)

---

## 📖 CONCEPT 3: INDEXES DEEP DIVE

### What is an Index Really?

An index is a **separate data structure** that points to rows in your table.

**Without Index:**
```
Table: [Row1] [Row2] [Row3] [Row4] [Row5] ...
Query: "Find age=30" → Scan every row (slow)
```

**With Index:**
```
Index: 25→Row2, 28→Row1, 30→Row4, 30→Row5, 35→Row3
Query: "Find age=30" → Look up index → Jump to Row4, Row5 (fast)
```

---

### B-Tree Indexes (Most Common)

**Structure:**
```
               [50]
              /    \
          [25]      [75]
         /   \      /   \
      [10] [40]  [60] [90]
       |    |     |    |
     Rows Rows  Rows Rows
```

**How It Works:**
1. Start at root node
2. Navigate down tree (few disk reads)
3. Find leaf node with actual row pointers
4. Fetch rows from table

**Best For:**
- Equality: `WHERE age = 30`
- Range: `WHERE age BETWEEN 25 AND 35`
- Sorting: `ORDER BY age`
- Prefix matching: `WHERE name LIKE 'Alice%'`

**Not Good For:**
- Suffix matching: `WHERE name LIKE '%Smith'`
- Function calls: `WHERE LOWER(name) = 'alice'`
- Negative conditions: `WHERE age != 30`

---

### Hash Indexes

**How It Works:**
- Hash function maps values to buckets
- Super fast for equality (`WHERE id = 123`)
- Useless for ranges (`WHERE id > 100`)

**When to Use:**
- Equality lookups only
- Rarely used (B-tree usually better)

---

### GIN and GiST Indexes

**GIN (Generalized Inverted Index):**
- For array, jsonb, full-text search
- Example: `WHERE tags @> ARRAY['postgres']`

**GiST (Generalized Search Tree):**
- For geometric data, ranges
- Example: PostGIS geospatial queries

---

### Index Best Practices

#### 1. Index Selectivity

**High Selectivity = Good Index:**
```sql
-- Good: user_id is unique
CREATE INDEX ON orders(user_id);

-- Bad: status has only 3 values (pending, shipped, delivered)
CREATE INDEX ON orders(status);  -- Probably not worth it
```

**Check selectivity:**
```sql
SELECT 
    COUNT(DISTINCT column_name) * 100.0 / COUNT(*) AS selectivity_pct
FROM table_name;
-- >10% selectivity usually worth indexing
```

#### 2. Composite Indexes

**Order matters!**
```sql
CREATE INDEX ON users(age, name);
-- Good for:
--   WHERE age = 30 AND name = 'Alice'
--   WHERE age = 30
-- NOT good for:
--   WHERE name = 'Alice'  (doesn't use index)
```

**Rule:** Most selective column first.

#### 3. Covering Indexes (Index-Only Scans)

```sql
-- Query
SELECT name, email FROM users WHERE age > 25;

-- Regular index (needs table lookup)
CREATE INDEX ON users(age);

-- Covering index (no table lookup!)
CREATE INDEX ON users(age, name, email);
-- Or use INCLUDE:
CREATE INDEX ON users(age) INCLUDE (name, email);
```

**Why It's Fast:**
- Reads only index pages
- No random access to table pages
- Can be 10x faster

#### 4. Partial Indexes

**Index only what you query:**
```sql
-- If you only query active users
CREATE INDEX ON users(email) WHERE status = 'active';
-- Smaller index → faster queries → less storage
```

#### 5. Functional Indexes

```sql
-- Query
SELECT * FROM users WHERE LOWER(email) = 'alice@example.com';

-- Regular index doesn't help
CREATE INDEX ON users(email);  -- Not used

-- Functional index
CREATE INDEX ON users(LOWER(email));  -- Used!
```

---

### When NOT to Use Indexes

❌ **Small tables (<1000 rows)**
- Sequential scan is faster
- Index overhead not worth it

❌ **Columns with low selectivity**
- boolean flags
- enums with few values

❌ **Heavy write workloads**
- Every INSERT/UPDATE must update indexes
- Too many indexes → slow writes

❌ **Columns never used in WHERE/JOIN/ORDER BY**

---

## 📖 CONCEPT 4: MVCC AND TRANSACTIONS

### ACID Properties

**Atomicity:** All or nothing
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- Both happen or neither
```

**Consistency:** Database stays valid
```sql
-- CHECK constraint ensures balance >= 0
UPDATE accounts SET balance = -100 WHERE id = 1;
-- ERROR: violates check constraint
```

**Isolation:** Transactions don't interfere
```sql
-- Transaction 1
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- Sees 100

-- Transaction 2 (concurrent)
BEGIN;
UPDATE accounts SET balance = 200 WHERE id = 1;
COMMIT;

-- Back to Transaction 1
SELECT balance FROM accounts WHERE id = 1;  -- Still sees 100!
COMMIT;
```

**Durability:** Committed = permanent (WAL)

---

### Isolation Levels

**Read Uncommitted:** (Postgres doesn't support this)
- Can see uncommitted changes
- Dirty reads possible

**Read Committed:** (Postgres default)
- Only sees committed changes
- Each statement sees latest commits

**Repeatable Read:**
- Sees snapshot at transaction start
- Same query returns same results

**Serializable:**
- Strictest level
- Transactions appear to run serially
- May get serialization errors

**Example:**
```sql
-- Session 1
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;  -- Returns 100

-- Session 2
UPDATE accounts SET balance = 200 WHERE id = 1;
COMMIT;

-- Back to Session 1
SELECT balance FROM accounts WHERE id = 1;  -- Still returns 100!
COMMIT;
```

---

### Common Concurrency Issues

**1. Dirty Reads:** Reading uncommitted data
- Prevented in Postgres (minimum Read Committed)

**2. Non-Repeatable Reads:** Row changes between reads
- Prevented with Repeatable Read or Serializable

**3. Phantom Reads:** New rows appear between reads
- Prevented with Repeatable Read or Serializable

**4. Lost Updates:**
```sql
-- Transaction 1
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- 100
-- ... do some calculation ...
UPDATE accounts SET balance = 150 WHERE id = 1;

-- Transaction 2 (concurrent)
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- 100
UPDATE accounts SET balance = 200 WHERE id = 1;  -- Overwrites T1!
COMMIT;

-- Transaction 1
COMMIT;  -- Lost the update from T2!
```

**Solution: SELECT FOR UPDATE**
```sql
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;  -- Locks row
UPDATE accounts SET balance = 150 WHERE id = 1;
COMMIT;
```

---

## 📖 CONCEPT 5: EXPLAIN ANALYZE MASTERY

### The Power of EXPLAIN

**Basic EXPLAIN:** Shows estimated plan
```sql
EXPLAIN SELECT * FROM users WHERE age > 25;
```

**EXPLAIN ANALYZE:** Runs query and shows actual vs estimated
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE age > 25;
```

**EXPLAIN with options:**
```sql
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) 
SELECT * FROM users WHERE age > 25;
```

---

### Reading EXPLAIN Output

```
Limit  (cost=0.42..1.67 rows=10 width=64) (actual time=0.028..0.042 rows=10 loops=1)
  ->  Index Scan using users_age_idx on users  (cost=0.42..187.54 rows=1500 width=64) (actual time=0.026..0.038 rows=10 loops=1)
        Index Cond: (age > 25)
Planning Time: 0.123 ms
Execution Time: 0.068 ms
```

**Key Metrics:**
- `cost=0.42..187.54` - Estimated cost range
- `rows=1500` - Estimated rows
- `actual time=0.026..0.038` - Actual time in ms
- `rows=10` - Actual rows
- `loops=1` - How many times this node executed

**Red Flags:**
- ❌ Estimated rows vs actual rows very different → bad statistics
- ❌ Seq Scan on large table
- ❌ Sort with large number of rows → increase work_mem
- ❌ Nested Loop with large outer → consider hash join

---

### Common Optimization Patterns

**Problem: Sequential Scan on Large Table**
```
Seq Scan on users  (cost=0.00..18755.00 rows=100000 width=64)
  Filter: (age > 25)
```
**Solution:** Add index on age
```sql
CREATE INDEX idx_users_age ON users(age);
```

**Problem: Index Not Used**
```
Seq Scan on users  (cost=0.00..180.00 rows=5000 width=64)
  Filter: (lower(email) = 'alice@example.com')
```
**Solution:** Functional index
```sql
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
```

**Problem: Slow Sort**
```
Sort  (cost=45234.23..47734.23 rows=1000000 width=64)
  Sort Key: created_at
  Sort Method: external merge  Disk: 68432kB
```
**Solution:** Increase work_mem OR add index
```sql
SET work_mem = '256MB';
-- Or
CREATE INDEX idx_users_created_at ON users(created_at);
```

---

## 💻 EXERCISES

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

### Exercise 2: Query Optimization

Given this slow query:

```sql
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
AND o.status = 'completed'
GROUP BY u.id, u.name
ORDER BY order_count DESC
LIMIT 10;
```

**Tasks:**
1. Run EXPLAIN ANALYZE (before any changes)
2. Identify the bottleneck
3. Add appropriate indexes
4. Rewrite query if needed
5. Run EXPLAIN ANALYZE again
6. Document the improvement

---

### Exercise 3: EXPLAIN Analysis

Run these queries and analyze their plans:

```sql
-- Query A
EXPLAIN (ANALYZE, BUFFERS) 
SELECT * FROM users WHERE email = 'specific@email.com';

-- Query B
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM users WHERE email LIKE '%example.com';

-- Query C
EXPLAIN (ANALYZE, BUFFERS)
SELECT COUNT(*) FROM users WHERE status = 'active';

-- Query D
EXPLAIN (ANALYZE, BUFFERS)
SELECT u.*, o.total 
FROM users u 
JOIN orders o ON u.id = o.user_id
WHERE o.total > 100
ORDER BY o.total DESC
LIMIT 10;
```

**For each query, answer:**
1. What type of scan was used?
2. Were indexes used?
3. What was the actual time?
4. How would you optimize it?

---

### Exercise 4: Transaction Isolation

Test isolation levels:

```sql
-- Terminal 1
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM accounts WHERE id = 1;
-- Leave transaction open

-- Terminal 2
UPDATE accounts SET balance = 500 WHERE id = 1;
COMMIT;

-- Back to Terminal 1
SELECT balance FROM accounts WHERE id = 1;
-- What do you see?
COMMIT;

-- Repeat with REPEATABLE READ isolation level
```

**Questions:**
1. What balance did you see in each case?
2. Why the difference?
3. When would you use each isolation level?

---

## 📝 QUIZ (Pass with 80%+)

### Question 1
What is the default page size in PostgreSQL?
- A) 4KB
- B) 8KB ✓
- C) 16KB
- D) 32KB

### Question 2
What does MVCC stand for?
- A) Multi-Version Concurrency Control ✓
- B) Multiple Virtual Cache Control
- C) Managed Version Change Control
- D) Memory Version Cache Control

### Question 3
When does PostgreSQL create a new row version?
- A) Only on INSERT
- B) On INSERT and DELETE
- C) On INSERT, UPDATE, and DELETE ✓
- D) Never, it updates in place

### Question 4
What does VACUUM do?
- A) Deletes rows
- B) Marks dead tuples as reusable ✓
- C) Compresses the database
- D) Backs up the database

### Question 5
What does "cost" in EXPLAIN output represent?
- A) Actual execution time in milliseconds
- B) Abstract units for comparison ✓
- C) Money spent on AWS
- D) Number of CPU cycles

### Question 6
Which scan type reads every row in a table?
- A) Index Scan
- B) Bitmap Heap Scan
- C) Sequential Scan ✓
- D) Index Only Scan

### Question 7
What is a covering index?
- A) An index that covers multiple tables
- B) An index that contains all columns needed by a query ✓
- C) An index created on multiple columns
- D) An index that's hidden

### Question 8
When should you run ANALYZE?
- A) Never, it's automatic
- B) After bulk INSERT/UPDATE/DELETE ✓
- C) Only when the database crashes
- D) Every hour

### Question 9
What is the default isolation level in PostgreSQL?
- A) Read Uncommitted
- B) Read Committed ✓
- C) Repeatable Read
- D) Serializable

### Question 10
What does SELECT FOR UPDATE do?
- A) Allows selecting data for updates
- B) Locks the row for updates ✓
- C) Prepares an UPDATE statement
- D) Nothing, it's deprecated

**Answers at bottom of quiz file**

---

## 🎯 PROJECT: DATABASE PERFORMANCE DETECTIVE

### Setup

Create a "slow" database with performance issues:

```sql
-- Create tables
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50),
    country VARCHAR(50),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date TIMESTAMP,
    total DECIMAL(10,2),
    status VARCHAR(20)
);

CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER,
    product_name VARCHAR(200),
    quantity INTEGER,
    price DECIMAL(10,2)
);

-- Insert data
INSERT INTO customers (name, email, city, country)
SELECT 
    'Customer ' || gs,
    'customer' || gs || '@example.com',
    (ARRAY['New York', 'London', 'Tokyo', 'Paris', 'Berlin'])[floor(random() * 5 + 1)],
    (ARRAY['USA', 'UK', 'Japan', 'France', 'Germany'])[floor(random() * 5 + 1)]
FROM generate_series(1, 50000) gs;

INSERT INTO orders (customer_id, order_date, total, status)
SELECT 
    floor(random() * 50000 + 1)::INTEGER,
    NOW() - (random() * interval '365 days'),
    (random() * 1000)::DECIMAL(10,2),
    (ARRAY['pending', 'completed', 'shipped', 'cancelled'])[floor(random() * 4 + 1)]
FROM generate_series(1, 200000);

INSERT INTO order_items (order_id, product_name, quantity, price)
SELECT 
    floor(random() * 200000 + 1)::INTEGER,
    'Product ' || floor(random() * 100 + 1),
    floor(random() * 10 + 1)::INTEGER,
    (random() * 100)::DECIMAL(10,2)
FROM generate_series(1, 500000);
```

---

### 5 Slow Queries to Debug

**Query 1: Customer Lookup**
```sql
SELECT * FROM customers WHERE email = 'customer12345@example.com';
```
**Problem:** ?
**Solution:** ?

**Query 2: Recent Orders**
```sql
SELECT * FROM orders 
WHERE order_date > NOW() - interval '30 days'
ORDER BY order_date DESC;
```
**Problem:** ?
**Solution:** ?

**Query 3: Order Summary**
```sql
SELECT 
    c.name,
    c.email,
    COUNT(o.id) as order_count,
    SUM(o.total) as total_spent
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE c.country = 'USA'
GROUP BY c.id, c.name, c.email
ORDER BY total_spent DESC
LIMIT 10;
```
**Problem:** ?
**Solution:** ?

**Query 4: Product Sales**
```sql
SELECT 
    oi.product_name,
    SUM(oi.quantity) as total_quantity,
    SUM(oi.quantity * oi.price) as revenue
FROM order_items oi
JOIN orders o ON oi.order_id = o.id
WHERE o.status = 'completed'
GROUP BY oi.product_name
ORDER BY revenue DESC;
```
**Problem:** ?
**Solution:** ?

**Query 5: Customer Search**
```sql
SELECT * FROM customers WHERE name LIKE '%Smith%';
```
**Problem:** ?
**Solution:** ?

---

### Deliverables

Create these files in your project folder:

**1. `analysis.md`** - Your investigation report
```markdown
# Database Performance Analysis

## Query 1: Customer Lookup
### Original Performance
- Execution time: X ms
- Plan: [paste EXPLAIN ANALYZE output]
- Problem identified: No index on email

### After Optimization
- Execution time: Y ms
- Plan: [paste EXPLAIN ANALYZE output]
- Improvement: X% faster

### Solution Applied
```sql
CREATE INDEX idx_customers_email ON customers(email);
```

[Repeat for all 5 queries]
```

**2. `optimized-queries.sql`** - Your optimized versions
**3. `indexes-created.sql`** - All indexes you added
**4. `README.md`** - Project overview and learnings

---

## 🎤 INTERVIEW PREP

### Questions You Must Be Able to Answer

**1. Explain how PostgreSQL stores data on disk.**
*Your answer should mention: pages, tuples, MVCC, TOAST*

**2. What is MVCC and why does it matter?**
*Your answer should mention: multiple row versions, no read locks, dead tuples, VACUUM*

**3. Walk me through what happens when you run a SELECT query.**
*Your answer should mention: parser, planner, executor, query plan*

**4. How do you debug a slow query?**
*Your answer should mention: EXPLAIN ANALYZE, check for seq scans, check statistics, add indexes*

**5. When would you NOT add an index?**
*Your answer should mention: small tables, low selectivity, heavy writes*

**6. What's the difference between VACUUM and VACUUM FULL?**
*Your answer should mention: dead tuple cleanup vs table rewrite, locking*

**7. Explain the different types of index scans.**
*Your answer should mention: seq scan, index scan, bitmap heap scan, index-only scan*

**8. What isolation levels does PostgreSQL support?**
*Your answer should mention: Read Committed (default), Repeatable Read, Serializable*

**9. How do you prevent lost updates?**
*Your answer should mention: SELECT FOR UPDATE, optimistic locking, Serializable isolation*

**10. What's a covering index?**
*Your answer should mention: index contains all columns needed, no table lookup, index-only scan*

---

## ✅ WEEK 1 CHECKLIST

**By Sunday March 2nd, you must have:**

- [ ] Read all 5 concepts
- [ ] Completed all 4 exercises
- [ ] Passed quiz with 80%+
- [ ] Completed Database Performance Detective project
- [ ] Analyzed all 5 slow queries
- [ ] Created optimization report
- [ ] Pushed everything to GitHub
- [ ] Started blog post draft
- [ ] Reviewed all 10 interview questions
- [ ] Posted update in your Notion tracker

**GitHub should contain:**
```
week-01-databases/
├── 01-concepts/ (your notes on each concept)
├── 02-exercises/ (completed exercises)
├── 03-quiz/ (your answers)
├── 04-project/
│   ├── setup.sql
│   ├── slow-queries.sql
│   ├── analysis.md ⭐
│   ├── optimized-queries.sql ⭐
│   ├── indexes-created.sql ⭐
│   └── README.md ⭐
└── 05-interview-prep/ (your answers to 10 questions)
```

---

## 📚 RESOURCES

### Must Read
- [PostgreSQL Storage Documentation](https://www.postgresql.org/docs/current/storage.html)
- [Use The Index, Luke](https://use-the-index-luke.com/) - Free book on indexes

### Must Watch
- [Hussein Nasser - PostgreSQL Internals](https://www.youtube.com/watch?v=Q56kljmIN14)
- [Hussein Nasser - MVCC Explained](https://www.youtube.com/watch?v=bENkqQxOPPw)

### Optional Deep Dives
- [Postgres EXPLAIN Visualizer](https://explain.dalibo.com/)
- [pgMustard - EXPLAIN Analysis Tool](https://www.pgmustard.com/)

---

## 💬 SUPPORT

**Stuck? Check:**
1. PostgreSQL docs (always start here)
2. Stack Overflow (search your error)
3. Come back to me with specific questions

**When asking for help, provide:**
- What you tried
- Error message (full text)
- Your EXPLAIN output
- Your hypothesis on what's wrong

---

**Now GO! Start reading Concept 1 and don't stop until Sunday! 🚀**

**Next check-in: March 2nd - I'll review your project!**