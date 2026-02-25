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