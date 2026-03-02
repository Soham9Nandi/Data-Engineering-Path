
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
