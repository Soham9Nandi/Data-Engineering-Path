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
