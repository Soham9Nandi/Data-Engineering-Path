# How PostgreSQL Stores Data on Disk

## The "Why" Behind Understanding Storage

You've been writing SQL queries, but do you know what happens when you run `SELECT * FROM users WHERE id = 123`? Understanding storage is the difference between "it works" and "I know why it's slow."

---

## 1. Pages: The Fundamental Unit

PostgreSQL doesn't read individual rows from disk. It reads in **pages** (also called blocks).

**Key Facts:**
- Default page size: **8KB**
- A page contains multiple rows (tuples)
- PostgreSQL reads entire pages into memory, even if you need just one row

**Example:**
```sql
-- Your users table has 1000 rows
-- Each row is ~200 bytes
-- That's ~125 rows per 8KB page
-- So 1000 rows = ~8 pages on disk
```

**Why This Matters:**
- If your row spans multiple pages (e.g., large TEXT fields), queries get slower
- Sequential scans read pages in order (fast for disks)
- Random access jumps between pages (slow)

---

## 2. How Rows (Tuples) Are Stored

Each row in PostgreSQL has:
1. **Row header** (~23 bytes): metadata about the row
2. **Actual data**: your columns
3. **Padding**: alignment to optimize CPU access

**Example Table:**
```sql
CREATE TABLE users (
    id INTEGER,           -- 4 bytes
    name VARCHAR(50),     -- variable, but let's say 20 bytes
    email VARCHAR(100),   -- variable, let's say 30 bytes
    created_at TIMESTAMP  -- 8 bytes
);
```

**Total per row:** ~23 (header) + 4 + 20 + 30 + 8 = **~85 bytes**

**This means:** ~95 rows per 8KB page

---

## 3. MVCC: Multi-Version Concurrency Control

**The Problem:** How do multiple users read/write simultaneously without locking?

**PostgreSQL's Solution:** Keep multiple versions of each row.

### How It Works:

When you UPDATE a row, PostgreSQL:
1. Doesn't actually update it in place
2. Creates a NEW version of the row
3. Marks the old version as "dead" (eventually)

**Example:**
```sql
-- Initial state
INSERT INTO users VALUES (1, 'Alice', 'alice@example.com');
-- Row version 1 created

-- Update
UPDATE users SET name = 'Alice Smith' WHERE id = 1;
-- Row version 2 created
-- Row version 1 still exists but marked "old"

-- Another update
UPDATE users SET email = 'alice.smith@example.com' WHERE id = 1;
-- Row version 3 created
-- Row versions 1 and 2 are now "dead tuples"
```

**Why This Matters:**
- Old row versions pile up ("bloat")
- VACUUM cleans up dead tuples
- Frequent updates = more bloat = slower queries
- This is why you see "dead tuples" in production databases

**Check bloat:**
```sql
SELECT 
    schemaname, 
    tablename, 
    n_live_tup AS live_rows, 
    n_dead_tup AS dead_rows,
    ROUND(n_dead_tup * 100.0 / NULLIF(n_live_tup, 0), 2) AS bloat_percentage
FROM pg_stat_user_tables
WHERE n_dead_tup > 0
ORDER BY n_dead_tup DESC;
```

---

## 4. Transaction IDs and Visibility

Every row has transaction IDs that determine **who can see it**.

**Key Fields:**
- `xmin`: Transaction that created this row version
- `xmax`: Transaction that deleted/updated this row version

**Example:**
```sql
-- Transaction 100 inserts a row
BEGIN; -- Transaction ID = 100
INSERT INTO users VALUES (1, 'Bob');
COMMIT;
-- Row has xmin=100, xmax=0 (not deleted)

-- Transaction 101 tries to read
BEGIN; -- Transaction ID = 101
SELECT * FROM users WHERE id = 1;
-- Sees the row because 101 > 100 (row is visible)

-- Transaction 102 deletes
BEGIN; -- Transaction ID = 102
DELETE FROM users WHERE id = 1;
COMMIT;
-- Row now has xmin=100, xmax=102

-- Transaction 103 tries to read
BEGIN; -- Transaction ID = 103
SELECT * FROM users WHERE id = 1;
-- Doesn't see it because 103 > 102 (row is deleted)
```

**Why This Matters:**
- Explains why long-running transactions are bad (they prevent VACUUM)
- Explains "snapshot too old" errors
- Explains why reads don't block writes (MVCC magic)

---

## 5. The Write-Ahead Log (WAL)

**The Problem:** If Postgres crashes mid-write, data could be corrupted.

**Solution:** Write changes to a log first, then to actual data files.

**How It Works:**
1. You run `INSERT INTO users ...`
2. Change is written to WAL (sequential write, fast)
3. Change is written to memory (in shared buffers)
4. Later, background process writes to actual data files

**Why This Matters:**
- WAL files pile up if not archived
- WAL is used for replication (streaming replication reads WAL)
- WAL is used for point-in-time recovery
- Heavy write workloads = lots of WAL = disk I/O

**Check WAL usage:**
```sql
SELECT pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), '0/0'));
```

---

## 6. TOAST: Oversized Attributes

**Problem:** What if a column value is bigger than a page (8KB)?

**Solution:** TOAST (The Oversized-Attribute Storage Technique)

**How It Works:**
- Large values (>~2KB) are compressed and/or stored out-of-line
- The main table stores a pointer to the TOASTed value
- TOAST data is stored in a separate table (`pg_toast.pg_toast_<table_oid>`)

**Example:**
```sql
CREATE TABLE documents (
    id INTEGER,
    content TEXT  -- Could be megabytes
);

INSERT INTO documents VALUES (1, repeat('A', 1000000)); -- 1MB of text
-- This gets TOASTed
```

**Why This Matters:**
- `SELECT *` on tables with large TEXT/BYTEA columns is slow
- Only select columns you need
- TOASTed values are fetched separately (extra I/O)

**Check TOAST usage:**
```sql
SELECT 
    n.nspname AS schema,
    c.relname AS table,
    pg_size_pretty(pg_total_relation_size(c.oid)) AS total_size,
    pg_size_pretty(pg_relation_size(c.oid)) AS table_size,
    pg_size_pretty(pg_total_relation_size(c.oid) - pg_relation_size(c.oid)) AS toast_size
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind = 'r'
AND pg_total_relation_size(c.oid) > 0
ORDER BY pg_total_relation_size(c.oid) DESC
LIMIT 10;
```

---

## Key Takeaways

1. **Pages are 8KB** - understand this and you understand performance
2. **MVCC creates row versions** - updates don't update in place
3. **Dead tuples cause bloat** - VACUUM is essential
4. **Transaction IDs control visibility** - this is how reads don't block writes
5. **WAL ensures durability** - but can pile up
6. **TOAST handles large values** - avoid `SELECT *` with TEXT columns

---

## Interview Questions You Can Now Answer

**Q: "Why is my database slow after lots of updates?"**
A: "Updates in PostgreSQL create new row versions due to MVCC. The old versions become dead tuples, causing bloat. You need to run VACUUM to reclaim space. I'd check `pg_stat_user_tables` to see the dead tuple percentage."

**Q: "What's the difference between VACUUM and VACUUM FULL?"**
A: "VACUUM marks dead tuples as reusable but doesn't shrink the table. VACUUM FULL actually rewrites the table to reclaim space, but it locks the table. For large tables, VACUUM FULL can take hours."

**Q: "Why does SELECT * slow down when I add a large TEXT column?"**
A: "PostgreSQL TOASTs large values (>~2KB) to separate storage. When you SELECT *, it fetches all TOASTed values even if you don't need them. Better to select only the columns you need."

---

## Required Reading

1. **PostgreSQL Documentation:** [Database Physical Storage](https://www.postgresql.org/docs/current/storage.html)
2. **Article:** ["Understanding MVCC in PostgreSQL"](https://www.postgresql.org/docs/current/mvcc.html)
3. **YouTube:** Search "Hussein Nasser PostgreSQL MVCC" (15 min video)

---

## Practice Exercise

Run these queries on any PostgreSQL database you have access to:

```sql
-- See your table sizes and bloat
SELECT * FROM pg_stat_user_tables;

-- See dead tuple ratios
SELECT 
    relname,
    n_live_tup,
    n_dead_tup,
    ROUND(n_dead_tup * 100.0 / NULLIF(n_live_tup, 0), 2) AS bloat_pct
FROM pg_stat_user_tables
WHERE n_dead_tup > 0;

-- See TOAST usage
SELECT 
    c.relname,
    pg_size_pretty(pg_total_relation_size(c.oid)) AS total_size,
    pg_size_pretty(pg_relation_size(c.oid)) AS table_size
FROM pg_class c
WHERE c.relkind = 'r'
ORDER BY pg_total_relation_size(c.oid) DESC
LIMIT 10;
```

Take notes on what you find. You'll need this for the project!

---

**Next:** Move on to Concept 2 - Query Execution Internals