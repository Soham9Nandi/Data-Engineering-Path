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
