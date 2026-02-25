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
