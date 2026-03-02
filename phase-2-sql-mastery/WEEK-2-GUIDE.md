# WEEK 2: SQL MASTERY (BEYOND LEETCODE)
## Complete Learning Package

**Deadline: March 9th, 2025 (Sunday)**

---

## 📅 DAILY SCHEDULE

### Monday (Mar 3)
- Read Concept 1: Window Functions Deep Dive
- Read Concept 2: Recursive CTEs
- Watch: recommended videos (links in resources)

### Tuesday (Mar 4)
- Read Concept 3: Subquery vs JOIN vs CTE — Performance Reality
- Read Concept 4: UNION vs UNION ALL
- Complete Exercise 1 & 2

### Wednesday (Mar 5)
- Read Concept 5: Query Optimization Patterns
- Complete Exercise 3 & 4
- Take the quiz

### Thursday (Mar 6)
- Start project: SQL Performance Lab
- Set up the 10M row dataset
- Solve first 4 problems + measure performance

### Friday (Mar 7)
- Solve remaining 6 problems
- Run EXPLAIN ANALYZE on each
- Document approach + timings

### Saturday (Mar 8)
- Write optimization report
- Compare approaches (subquery vs CTE vs JOIN)
- Document findings

### Sunday (Mar 9)
- Polish project README
- Review interview prep questions
- Write/finish blog post draft
- Push everything to GitHub by EOD

---

## 📖 CONCEPT 1: WINDOW FUNCTIONS DEEP DIVE

### What Are Window Functions Really?

A window function performs a calculation **across a set of rows related to the current row** — without collapsing them into one row like GROUP BY does.

```sql
-- GROUP BY collapses rows (you lose detail)
SELECT department, AVG(salary) FROM employees GROUP BY department;

-- Window function keeps all rows AND adds the aggregate
SELECT 
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
```

**The key insight:** Window functions let you answer questions like *"how does this row compare to its group?"* — impossible with GROUP BY alone.

---

### Anatomy of a Window Function

```sql
function_name() OVER (
    PARTITION BY column   -- Define the "window" (like GROUP BY)
    ORDER BY column       -- Order within the window
    ROWS/RANGE BETWEEN ... AND ...  -- Frame specification
)
```

---

### ROW_NUMBER, RANK, DENSE_RANK

These three look similar but behave differently on ties:

```sql
SELECT 
    name,
    score,
    ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num,
    RANK()       OVER (ORDER BY score DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM scores;
```

**Result:**
```
name    score  row_num  rank  dense_rank
Alice   100    1        1     1
Bob     100    2        1     1
Carol   90     3        3     2    ← RANK skips 2, DENSE_RANK doesn't
Dave    80     4        4     3
```

**When to use which:**
- `ROW_NUMBER()` → deduplication, pagination (always unique)
- `RANK()` → competition rankings (Olympic style: 1,1,3)
- `DENSE_RANK()` → when you don't want gaps (1,1,2)

---

### LAG and LEAD

Access values from previous or next rows — no self-join needed.

```sql
SELECT 
    date,
    revenue,
    LAG(revenue, 1)  OVER (ORDER BY date) AS prev_day_revenue,
    LEAD(revenue, 1) OVER (ORDER BY date) AS next_day_revenue,
    revenue - LAG(revenue, 1) OVER (ORDER BY date) AS day_over_day_change
FROM daily_revenue;
```

**Real use case in fintech:** "Show each transaction and flag if it's more than 2x the user's previous transaction."

```sql
SELECT 
    user_id,
    transaction_id,
    amount,
    LAG(amount) OVER (PARTITION BY user_id ORDER BY created_at) AS prev_amount,
    CASE 
        WHEN amount > LAG(amount) OVER (PARTITION BY user_id ORDER BY created_at) * 2
        THEN 'FLAG'
        ELSE 'OK'
    END AS fraud_flag
FROM transactions;
```

---

### Running Totals and Moving Averages

```sql
-- Running total
SELECT 
    date,
    revenue,
    SUM(revenue) OVER (ORDER BY date) AS running_total
FROM daily_revenue;

-- 7-day moving average
SELECT 
    date,
    revenue,
    AVG(revenue) OVER (
        ORDER BY date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7d
FROM daily_revenue;

-- Month-to-date total
SELECT 
    date,
    revenue,
    SUM(revenue) OVER (
        PARTITION BY DATE_TRUNC('month', date)
        ORDER BY date
    ) AS mtd_revenue
FROM daily_revenue;
```

---

### Frame Specification (The Part Everyone Ignores)

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
-- Exactly 2 rows before + current row (count-based)

RANGE BETWEEN INTERVAL '7 days' PRECEDING AND CURRENT ROW
-- All rows within 7 days of current (value-based)

ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
-- All rows from start up to current (running total)

ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
-- All rows in partition (same as no frame spec)
```

**ROWS vs RANGE:**
- `ROWS` = physical rows (count)
- `RANGE` = logical range (value-based, handles duplicates differently)

---

### NTILE and Percentiles

```sql
-- Divide into quartiles
SELECT 
    customer_id,
    total_spent,
    NTILE(4) OVER (ORDER BY total_spent) AS quartile
FROM customers;

-- Top 10% of customers
SELECT * FROM (
    SELECT 
        customer_id,
        total_spent,
        NTILE(10) OVER (ORDER BY total_spent DESC) AS decile
    FROM customers
) t WHERE decile = 1;

-- PERCENTILE_CONT (continuous) vs PERCENTILE_DISC (discrete)
SELECT 
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary,
    PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary_disc
FROM employees;
```

---

### FIRST_VALUE, LAST_VALUE, NTH_VALUE

```sql
SELECT 
    department,
    name,
    salary,
    FIRST_VALUE(name) OVER (PARTITION BY department ORDER BY salary DESC) AS highest_earner,
    LAST_VALUE(name)  OVER (
        PARTITION BY department 
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_earner
FROM employees;
```

⚠️ **LAST_VALUE gotcha:** Without explicit frame spec, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` — so LAST_VALUE only looks up to the current row, not the end. Always specify the frame when using LAST_VALUE.

---

## 📖 CONCEPT 2: RECURSIVE CTEs

### What Is a Recursive CTE?

A way to query **hierarchical or graph-like data** by repeatedly applying a query to its own results.

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor member (base case)
    SELECT ...
    
    UNION ALL
    
    -- Recursive member (references cte_name)
    SELECT ... FROM cte_name JOIN ...
)
SELECT * FROM cte_name;
```

---

### The Classic: Org Chart / Hierarchy

```sql
-- Table: employees(id, name, manager_id)

WITH RECURSIVE org_chart AS (
    -- Anchor: start with the CEO (no manager)
    SELECT id, name, manager_id, 0 AS level, name::TEXT AS path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: find direct reports of each person
    SELECT 
        e.id, 
        e.name, 
        e.manager_id, 
        oc.level + 1,
        oc.path || ' → ' || e.name
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT 
    REPEAT('  ', level) || name AS hierarchy,
    level,
    path
FROM org_chart
ORDER BY path;
```

**Output:**
```
hierarchy          level  path
CEO                0      CEO
  CTO              1      CEO → CTO
    Engineer 1     2      CEO → CTO → Engineer 1
    Engineer 2     2      CEO → CTO → Engineer 2
  CFO              1      CEO → CFO
```

---

### Generating Series / Number Sequences

```sql
-- Generate dates for a date range
WITH RECURSIVE date_series AS (
    SELECT '2024-01-01'::DATE AS d
    UNION ALL
    SELECT d + 1 FROM date_series WHERE d < '2024-01-31'
)
SELECT d FROM date_series;
```

*(Note: PostgreSQL has `generate_series()` built-in, but this shows the pattern)*

---

### Graph Traversal: Finding All Paths

```sql
-- Find all ancestors of an employee
WITH RECURSIVE ancestors AS (
    SELECT id, name, manager_id, 1 AS depth
    FROM employees
    WHERE id = 42  -- Start from employee 42
    
    UNION ALL
    
    SELECT e.id, e.name, e.manager_id, a.depth + 1
    FROM employees e
    JOIN ancestors a ON e.id = a.manager_id
)
SELECT * FROM ancestors ORDER BY depth;
```

---

### Preventing Infinite Loops

Always include a termination condition:

```sql
WITH RECURSIVE traverse AS (
    SELECT id, parent_id, 1 AS depth, ARRAY[id] AS visited
    FROM nodes
    WHERE id = 1
    
    UNION ALL
    
    SELECT n.id, n.parent_id, t.depth + 1, t.visited || n.id
    FROM nodes n
    JOIN traverse t ON n.parent_id = t.id
    WHERE 
        n.id != ALL(t.visited)  -- ← Cycle detection
        AND t.depth < 100        -- ← Safety limit
)
SELECT * FROM traverse;
```

---

## 📖 CONCEPT 3: SUBQUERY vs JOIN vs CTE — PERFORMANCE REALITY

### The Three Approaches

```sql
-- Same question: "Get customers who placed orders over $500"

-- Approach 1: Subquery
SELECT * FROM customers
WHERE id IN (SELECT customer_id FROM orders WHERE total > 500);

-- Approach 2: JOIN
SELECT DISTINCT c.* FROM customers c
JOIN orders o ON c.id = o.customer_id
WHERE o.total > 500;

-- Approach 3: CTE
WITH big_orders AS (
    SELECT customer_id FROM orders WHERE total > 500
)
SELECT * FROM customers
WHERE id IN (SELECT customer_id FROM big_orders);
```

**Surprise:** In modern PostgreSQL, the query planner often produces **the same execution plan** for all three. Your choice matters more for readability than raw performance — until it doesn't.

---

### When Performance Actually Differs

**Correlated Subqueries — The Slow One:**

```sql
-- SLOW: Correlated subquery runs once per row
SELECT 
    c.name,
    (SELECT COUNT(*) FROM orders o WHERE o.customer_id = c.id) AS order_count
FROM customers c;

-- FAST: Single pass with JOIN + GROUP BY
SELECT 
    c.name,
    COUNT(o.id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name;
```

The correlated subquery executes N times (once per customer row). With 100k customers, that's 100k separate lookups.

---

### CTE Fence in PostgreSQL 12+

**Before PostgreSQL 12:** CTEs were "optimization fences" — the planner couldn't push predicates inside them.

**PostgreSQL 12+:** CTEs are inlined by default (treated like subqueries).

```sql
-- This used to be slow (pre-PG12): CTE wasn't optimized
WITH active_users AS (
    SELECT * FROM users WHERE status = 'active'
)
SELECT * FROM active_users WHERE country = 'USA';

-- PG12+ inlines it, equivalent to:
SELECT * FROM users WHERE status = 'active' AND country = 'USA';

-- Force the old behavior (create a barrier) with MATERIALIZED:
WITH active_users AS MATERIALIZED (
    SELECT * FROM users WHERE status = 'active'
)
SELECT * FROM active_users WHERE country = 'USA';
```

**When to use MATERIALIZED:** When the CTE result is expensive to compute and referenced multiple times in the query.

---

### The Decision Framework

```
Need to reuse a result multiple times?        → CTE
Hierarchical/recursive data?                  → Recursive CTE
Simple filter on another table?               → JOIN (usually fastest)
Correlated logic (per-row calculation)?       → Window function > subquery
One-off subquery in WHERE clause?             → Either, check EXPLAIN
```

---

## 📖 CONCEPT 4: UNION vs UNION ALL

### The Difference

```sql
-- UNION ALL: Returns all rows including duplicates (FAST)
SELECT id FROM table_a
UNION ALL
SELECT id FROM table_b;

-- UNION: Removes duplicates (SLOW - requires sort/hash)
SELECT id FROM table_a
UNION
SELECT id FROM table_b;
```

**How UNION removes duplicates:** It sorts or hashes the combined result set and removes matches — similar to `SELECT DISTINCT`. This is expensive on large datasets.

---

### Performance Test (What You'll Actually See)

```
UNION ALL on 1M rows each:    ~50ms
UNION on 1M rows each:        ~800ms  ← 16x slower
```

**Rule:** Always use `UNION ALL` unless you explicitly need deduplication. If you need deduplication, consider whether you can achieve it with a more targeted `WHERE` clause instead.

---

### Common Pattern: Combining Historical and Current Data

```sql
-- Common in data warehouses: current + archive tables
SELECT order_id, customer_id, total, created_at, 'current' AS source
FROM orders_current
WHERE created_at >= '2024-01-01'

UNION ALL

SELECT order_id, customer_id, total, created_at, 'archive' AS source
FROM orders_archive
WHERE created_at >= '2024-01-01';
```

---

### EXCEPT and INTERSECT (The Forgotten Ones)

```sql
-- EXCEPT: Rows in first query but not in second
SELECT customer_id FROM customers
EXCEPT
SELECT customer_id FROM orders;
-- Customers who have never ordered

-- INTERSECT: Rows in both queries
SELECT customer_id FROM newsletter_subscribers
INTERSECT
SELECT customer_id FROM paying_customers;
-- Subscribers who are also paying customers
```

Both `EXCEPT` and `INTERSECT` remove duplicates (like `UNION`). Use `EXCEPT ALL` and `INTERSECT ALL` for the `UNION ALL` equivalent.

---

## 📖 CONCEPT 5: QUERY OPTIMIZATION PATTERNS

### Pattern 1: Filter Early, Join Late

```sql
-- BAD: Join everything, then filter
SELECT c.name, o.total
FROM customers c
JOIN orders o ON c.id = o.customer_id
WHERE o.created_at > '2024-01-01'
AND c.country = 'USA';

-- BETTER: Pre-filter in subqueries (sometimes — check EXPLAIN)
SELECT c.name, o.total
FROM (SELECT * FROM customers WHERE country = 'USA') c
JOIN (SELECT * FROM orders WHERE created_at > '2024-01-01') o 
    ON c.id = o.customer_id;
```

*(Note: Modern planners often do this automatically — EXPLAIN will tell you if it matters)*

---

### Pattern 2: EXISTS vs IN vs JOIN for Existence Checks

```sql
-- Check if customer has any order

-- IN (can be slow with large subquery result)
SELECT * FROM customers 
WHERE id IN (SELECT customer_id FROM orders);

-- EXISTS (stops at first match — often faster)
SELECT * FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);

-- JOIN (fast but may duplicate rows without DISTINCT)
SELECT DISTINCT c.* FROM customers c
JOIN orders o ON c.id = o.customer_id;
```

**Rule of thumb:** `EXISTS` is usually fastest for existence checks because it short-circuits. But always verify with EXPLAIN.

---

### Pattern 3: Avoid Functions on Indexed Columns in WHERE

```sql
-- BAD: Function prevents index use
WHERE YEAR(created_at) = 2024
WHERE LOWER(email) = 'alice@example.com'
WHERE DATE(created_at) = '2024-01-01'

-- GOOD: Rewrite to use the index
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'
WHERE email = 'alice@example.com'  -- Store emails lowercased, or use functional index
WHERE created_at >= '2024-01-01' AND created_at < '2024-01-02'
```

---

### Pattern 4: COUNT(*) vs COUNT(column)

```sql
COUNT(*)        -- Counts all rows (fast)
COUNT(column)   -- Counts non-NULL values (slightly slower, scans for NULLs)
COUNT(DISTINCT column)  -- Counts unique non-NULL values (slowest)
```

Use `COUNT(*)` when you just need row count. Only use `COUNT(column)` when NULLs matter.

---

### Pattern 5: Pagination Done Right

```sql
-- BAD: Offset pagination (gets slower as offset increases)
SELECT * FROM orders ORDER BY id LIMIT 20 OFFSET 10000;
-- Has to scan and discard 10,000 rows!

-- GOOD: Keyset pagination (cursor-based)
SELECT * FROM orders 
WHERE id > 10000  -- last seen id
ORDER BY id 
LIMIT 20;
-- Jumps directly to id=10000 via index
```

This is huge in production systems. Offset pagination on page 500 is 500x slower than page 1.

---

## 💻 EXERCISES

### Exercise 1: Window Functions

Use the `test_users` table from Week 1 (or recreate it):

```sql
-- Add an orders table
CREATE TABLE test_orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER,
    amount DECIMAL(10,2),
    status VARCHAR(20),
    created_at TIMESTAMP DEFAULT NOW() - (random() * interval '365 days')
);

INSERT INTO test_orders (user_id, amount, status)
SELECT 
    (random() * 99999 + 1)::INTEGER,
    (random() * 500 + 10)::DECIMAL(10,2),
    CASE WHEN random() < 0.7 THEN 'completed' ELSE 'refunded' END
FROM generate_series(1, 500000);
```

**Problems to solve:**

1. For each user, show their orders with a running total of amount spent
2. Rank users by total spent — show ties correctly (use DENSE_RANK)
3. For each order, show the previous order amount for that user (LAG)
4. Calculate the 30-day moving average of daily revenue
5. Flag orders that are more than 2x the user's average order amount
6. Find the top 10% of users by total spend (NTILE)

**For each solution:** Run EXPLAIN ANALYZE and note the execution time.

---

### Exercise 2: Recursive CTE

```sql
-- Create a category hierarchy (e-commerce style)
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    parent_id INTEGER REFERENCES categories(id)
);

INSERT INTO categories (id, name, parent_id) VALUES
(1, 'Electronics', NULL),
(2, 'Clothing', NULL),
(3, 'Phones', 1),
(4, 'Laptops', 1),
(5, 'Mens', 2),
(6, 'Womens', 2),
(7, 'iPhone', 3),
(8, 'Android', 3),
(9, 'MacBook', 4),
(10, 'T-Shirts', 5);
```

**Problems to solve:**

1. Show the full hierarchy with indentation (like a tree)
2. Find all descendants of "Electronics"
3. Find the full path from root to any given category (e.g., "Electronics > Phones > iPhone")
4. Find the depth of each category in the tree
5. Bonus: Find all leaf nodes (categories with no children)

---

### Exercise 3: Performance Comparison

Use the 10M row table from the project setup. Run the same logical query three ways:

```sql
-- Question: Find customers who spent more than the average order amount

-- Approach A: Subquery
-- Approach B: CTE
-- Approach C: Window function

-- For each approach:
-- 1. Write the query
-- 2. Run EXPLAIN ANALYZE
-- 3. Record: planning time, execution time, scan types
-- 4. Explain WHY the plans differ (or don't)
```

---

### Exercise 4: UNION Performance

```sql
-- Create two tables
CREATE TABLE orders_2023 AS 
SELECT * FROM test_orders WHERE EXTRACT(YEAR FROM created_at) = 2023;

CREATE TABLE orders_2024 AS 
SELECT * FROM test_orders WHERE EXTRACT(YEAR FROM created_at) = 2024;

-- Test 1: UNION ALL vs UNION
EXPLAIN ANALYZE
SELECT user_id, amount FROM orders_2023
UNION ALL
SELECT user_id, amount FROM orders_2024;

EXPLAIN ANALYZE
SELECT user_id, amount FROM orders_2023
UNION
SELECT user_id, amount FROM orders_2024;

-- Questions:
-- 1. What extra step appears in the UNION plan?
-- 2. What is the time difference?
-- 3. How many duplicate rows were actually removed?
```

---

## 📝 QUIZ (Pass with 80%+)

### Question 1
What is the key difference between window functions and GROUP BY?

- A) Window functions are faster
- B) Window functions keep all rows while GROUP BY collapses them ✓
- C) GROUP BY supports more functions
- D) Window functions only work with ORDER BY

### Question 2
You have two users with the same score. `RANK()` returns 1,1,3. What does `DENSE_RANK()` return?

- A) 1, 1, 3
- B) 1, 2, 3
- C) 1, 1, 2 ✓
- D) 0, 0, 1

### Question 3
What does `LAG(amount, 2)` return?

- A) The next row's amount
- B) The amount 2 rows ahead
- C) The amount 2 rows behind ✓
- D) The average of 2 previous rows

### Question 4
What is the gotcha with `LAST_VALUE()`?

- A) It's not supported in PostgreSQL
- B) The default frame stops at the current row, not the partition end ✓
- C) It requires ORDER BY
- D) It only works with numeric columns

### Question 5
In PostgreSQL 12+, CTEs are:

- A) Always materialized (optimization fences)
- B) Inlined by default ✓
- C) Never inlined
- D) Deprecated in favor of subqueries

### Question 6
When does a correlated subquery run?

- A) Once for the entire query
- B) Once per unique value in the outer query
- C) Once per row in the outer query ✓
- D) In parallel with the outer query

### Question 7
Why is offset pagination slow?

- A) It requires a full table scan always
- B) It must scan and discard all rows before the offset ✓
- C) It doesn't support indexes
- D) It only works on small tables

### Question 8
`UNION` vs `UNION ALL` — which is faster and why?

- A) UNION, because it removes unnecessary rows
- B) UNION ALL, because it skips the deduplication step ✓
- C) They are identical in performance
- D) Depends on the database version

### Question 9
When should you use `EXISTS` instead of `IN`?

- A) Never, IN is always better
- B) When checking existence (it short-circuits at first match) ✓
- C) Only for small subqueries
- D) When the subquery returns NULL values

### Question 10
What does `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` mean in a window frame?

- A) The 6 rows after the current row
- B) All rows in the partition
- C) The current row and the 6 rows before it ✓
- D) Rows within a value range of 6

---

## 🎯 PROJECT: SQL PERFORMANCE LAB

### Setup: The 10M Row Dataset

```sql
-- Customers table
CREATE TABLE lab_customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    country VARCHAR(50),
    tier VARCHAR(20),  -- bronze, silver, gold, platinum
    created_at TIMESTAMP
);

-- Orders table
CREATE TABLE lab_orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    amount DECIMAL(10,2),
    status VARCHAR(20),
    product_category VARCHAR(50),
    created_at TIMESTAMP
);

-- Populate (this will take ~1-2 minutes)
INSERT INTO lab_customers (name, email, country, tier, created_at)
SELECT
    'Customer ' || gs,
    'customer' || gs || '@example.com',
    (ARRAY['USA', 'UK', 'Germany', 'France', 'Japan', 'Canada', 'Australia'])[floor(random() * 7 + 1)],
    (ARRAY['bronze', 'silver', 'gold', 'platinum'])[floor(random() * 4 + 1)],
    NOW() - (random() * interval '730 days')
FROM generate_series(1, 1000000) gs;

INSERT INTO lab_orders (customer_id, amount, status, product_category, created_at)
SELECT
    floor(random() * 1000000 + 1)::INTEGER,
    (random() * 2000 + 5)::DECIMAL(10,2),
    (ARRAY['completed', 'pending', 'refunded', 'cancelled'])[floor(random() * 4 + 1)],
    (ARRAY['Electronics', 'Clothing', 'Food', 'Books', 'Sports', 'Beauty'])[floor(random() * 6 + 1)],
    NOW() - (random() * interval '730 days')
FROM generate_series(1, 9000000) gs;

ANALYZE lab_customers;
ANALYZE lab_orders;
```

---

### 10 Problems to Solve Optimally

For each problem, you must:
1. Write the query
2. Run `EXPLAIN (ANALYZE, BUFFERS)` 
3. Record execution time
4. Explain your approach and why it's optimal

---

**Problem 1: Rank customers by total spend**
```sql
-- Find the top 100 customers by total completed order amount
-- Show: customer name, country, total_spent, rank
-- Only include customers with at least 5 completed orders
```

**Problem 2: Month-over-month revenue growth**
```sql
-- For each month in the last 12 months:
-- Show: month, revenue, previous month revenue, % growth
-- Hint: You'll need LAG()
```

**Problem 3: Customer tier analysis**
```sql
-- For each country + tier combination:
-- Show: avg order value, median order value, 90th percentile order value
-- Only for completed orders in 2024
```

**Problem 4: First and most recent order per customer**
```sql
-- For each customer, find:
-- Their first order date and amount
-- Their most recent order date and amount  
-- How many days between first and most recent order
-- Only for customers with 2+ orders
```

**Problem 5: Consecutive high-value orders**
```sql
-- Find customers who had 3 consecutive orders 
-- each above $1000 (completed status)
-- Hint: You'll need ROW_NUMBER() and LAG()
```

**Problem 6: Running total with reset**
```sql
-- For each customer, show their orders with:
-- A running total that resets each month
-- Hint: PARTITION BY customer_id, DATE_TRUNC('month', created_at)
```

**Problem 7: Category basket analysis**
```sql
-- Find pairs of product categories that are commonly ordered 
-- by the same customer (within the same month)
-- Show: category_1, category_2, co_occurrence_count
-- Sorted by co_occurrence_count DESC
```

**Problem 8: Cohort retention**
```sql
-- Monthly cohorts: group customers by the month they placed their FIRST order
-- For each cohort, show what % placed another order in months 1, 2, 3 after acquisition
```

**Problem 9: Outlier detection**
```sql
-- Find orders where the amount is more than 3 standard deviations 
-- from the customer's average order amount
-- Show: customer_id, order_id, amount, customer_avg, customer_stddev
```

**Problem 10: Optimization challenge**
```sql
-- This query is intentionally slow. Make it fast.
-- Record before and after execution times.

SELECT 
    c.name,
    c.country,
    (SELECT COUNT(*) FROM lab_orders o WHERE o.customer_id = c.id) AS total_orders,
    (SELECT SUM(amount) FROM lab_orders o WHERE o.customer_id = c.id AND status = 'completed') AS total_spent,
    (SELECT MAX(created_at) FROM lab_orders o WHERE o.customer_id = c.id) AS last_order_date
FROM lab_customers c
WHERE c.country = 'USA'
ORDER BY total_spent DESC NULLS LAST
LIMIT 20;
```

---

### Deliverables

```
week-02-sql-mastery/
├── 01-concepts/         (your notes)
├── 02-exercises/
│   ├── ex1-window-functions.sql
│   ├── ex2-recursive-cte.sql
│   ├── ex3-performance-comparison.sql
│   └── ex4-union-performance.sql
├── 03-quiz/             (your answers)
├── 04-project/
│   ├── setup.sql
│   ├── solutions.sql        ⭐ (all 10 problems)
│   ├── analysis.md          ⭐ (performance findings)
│   └── README.md            ⭐
└── 05-interview-prep/   (your answers to interview questions)
```

**`analysis.md` template:**
```markdown
# SQL Performance Lab — Analysis

## Problem 1: Rank Customers by Total Spend
### My Approach
[Explain your approach and why]

### Query
[Your query]

### EXPLAIN Output
[Paste explain analyze output]

### Execution Time: X ms
### Key Learnings
[What did you learn from this?]
```

---

## 🎤 INTERVIEW PREP

### Questions You Must Be Able to Answer

**1. What's the difference between ROW_NUMBER, RANK, and DENSE_RANK?**
*Your answer should mention: tie handling, skipping numbers (RANK), no gaps (DENSE_RANK)*

**2. How do window functions differ from GROUP BY?**
*Your answer should mention: rows preserved, aggregate calculated per row relative to window*

**3. When would you use a recursive CTE?**
*Your answer should mention: hierarchical data, org charts, graph traversal, infinite loop prevention*

**4. Explain the performance difference between UNION and UNION ALL.**
*Your answer should mention: deduplication cost, sort/hash operation, always use UNION ALL unless needed*

**5. What's a correlated subquery and why can it be slow?**
*Your answer should mention: executes per row, N executions, rewrite with JOIN or window function*

**6. How do you paginate efficiently on large tables?**
*Your answer should mention: avoid OFFSET, keyset/cursor pagination, WHERE id > last_seen*

**7. When does a CTE perform differently from a subquery in PostgreSQL?**
*Your answer should mention: PG12+ inlining, MATERIALIZED keyword, optimization fence*

**8. Write a query to find the top 3 customers per country by total spend.**
*Your answer should use: window function with PARTITION BY country, RANK or DENSE_RANK, filter on rank <= 3*

**9. How would you calculate a 7-day moving average in SQL?**
*Your answer should mention: window function, ROWS BETWEEN 6 PRECEDING AND CURRENT ROW*

**10. You have a slow query with multiple correlated subqueries. How do you fix it?**
*Your answer should mention: rewrite with JOINs + GROUP BY, or use window functions, show before/after EXPLAIN*

---

## ✅ WEEK 2 CHECKLIST

**By Sunday March 9th, you must have:**

- [ ] Read all 5 concepts
- [ ] Completed all 4 exercises
- [ ] Passed quiz with 80%+
- [ ] Set up the 10M row dataset
- [ ] Solved all 10 project problems
- [ ] Ran EXPLAIN ANALYZE on each problem
- [ ] Wrote analysis.md with findings
- [ ] Pushed everything to GitHub
- [ ] Started/finished blog post
- [ ] Reviewed all 10 interview questions
- [ ] Posted update in Notion tracker

---

## 📚 RESOURCES

### Must Read
- [PostgreSQL Window Functions](https://www.postgresql.org/docs/current/tutorial-window.html) — Official docs
- [Use The Index, Luke — Pagination](https://use-the-index-luke.com/sql/partial-results/fetch-next-page) — Keyset pagination explained
- [Modern SQL](https://modern-sql.com/feature/filter) — Window functions and modern SQL features

### Must Watch
- [Hussein Nasser - Window Functions](https://www.youtube.com/results?search_query=hussein+nasser+window+functions+postgresql) — Search his channel
- [CMU Database Course - Query Optimization](https://www.youtube.com/c/cmudatabasegroup) — Deeper dive if you want it

### Tools
- [Postgres EXPLAIN Visualizer](https://explain.dalibo.com/) — Paste your EXPLAIN output for a visual
- [pgMustard](https://www.pgmustard.com/) — EXPLAIN analysis with recommendations

---

**Now GO! Window functions will feel weird until they click — and then you'll use them everywhere. Push through. 🚀**

**Next check-in: March 9th — I'll review your project!**