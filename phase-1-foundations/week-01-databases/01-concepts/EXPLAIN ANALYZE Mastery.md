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
