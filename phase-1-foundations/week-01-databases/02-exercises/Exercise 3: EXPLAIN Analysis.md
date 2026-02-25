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
