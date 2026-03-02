# Your Custom Curriculum: "Deep Fundamentals + Interview Ready"

Based on everything you told me, here's what I'm building for you:

## Structure: 12 Weeks, Project-Based Learning

Not "2 hours of coding" - instead: "Complete Module X by Sunday"

Each week has: Concept deep-dive → Hands-on exercises → Mini-project → Interview prep

Deadline-driven (you said this works for you)

Mix of videos, docs, and coding (you need all three)

Interview questions mapped to each topic (so you learn depth AND interview patterns)

---

# Phase 1: Foundations (Weeks 1-4)

Goal: Fix the "I don't know why it works" problem

## Week 1: How Databases Actually Work

### Deep Dive:

- How does Postgres store data on disk? (pages, tuples, MVCC)
- What actually happens when you run a query? (parser → planner → executor)
- Indexes: B-trees, hash indexes, when to use what
- EXPLAIN ANALYZE: reading query plans like a pro

### Exercise Project: "Database Performance Detective"

Given: A slow database with 5 problematic queries

Task: Use EXPLAIN ANALYZE to diagnose and fix each one

Deliverable: Report explaining WHAT was wrong and WHY your fix worked

### Interview Prep:

- "Explain how an index works"
- "Why is this query slow?" (with query plan)
- "What's the difference between clustered and non-clustered indexes?"

---

## Week 2: SQL Mastery (Beyond Leetcode)

### Deep Dive:

- Window functions (the ones you don't know yet)
- Recursive CTEs (actually understanding them)
- Query optimization patterns
- When subquery vs JOIN vs CTE matters for performance
- UNION vs UNION ALL (with actual performance tests)

### Exercise Project: "SQL Performance Lab"

Dataset: 10M row table

10 problems that MUST be solved with optimal SQL

Each solution needs: query + execution time + explanation of approach

Learn by measuring: "This approach took 30s, this one took 2s - why?"

### Interview Prep:

- Common SQL interview questions (with depth)
- "Write a query to find X" → "Now optimize it" → "Explain your choices"

---

## Week 3: Python for Data Engineering (Not Just Syntax)

### Deep Dive:

- Context managers (with statement - how and why)
- Generators and memory efficiency (why this matters for big data)
- Decorators (used everywhere in frameworks)
- Error handling patterns (not just try/except, but proper error design)
- Working with large files (chunking, streaming, memory management)

### Exercise Project: "Build a Robust ETL Script"

- Extract from API with pagination and rate limiting
- Transform 5GB CSV in chunks (can't load in memory)
- Load to database in batches with proper error handling
- Add logging, retries, and progress tracking
- Write unit tests for each component

### Interview Prep:

- "How do you handle a file that's too large for memory?"
- "Explain decorators and give an example"
- Live coding: "Write a function that processes this data..."

---

## Week 4: Data Modeling Fundamentals

### Deep Dive:

- Star schema vs Snowflake schema (with real examples)
- Slowly Changing Dimensions (SCD Type 1, 2, 3)
- Fact vs Dimension tables (and why this matters)
- Normalization vs Denormalization (trade-offs)
- Data Vault modeling (basics - it's coming up in interviews)

### Exercise Project: "Design a Data Warehouse"

Scenario: E-commerce company needs analytics

Design: Star schema for sales, customers, products

Implement: Create tables, write ETL, handle SCDs

Deliverable: ERD + SQL scripts + documentation of design choices

### Interview Prep:

- "Design a schema for X use case"
- "How would you handle historical changes in dimension tables?"
- "Explain your modeling approach and why"

---

# Phase 2: Core DE Skills (Weeks 5-8)

Goal: Build production-quality data pipelines with depth

## Week 5: Airflow Deep Dive

### Deep Dive:

- How Airflow actually works (scheduler, executor, metadata DB)
- Task instances, DAG runs, execution dates (the confusing parts)
- Backfilling vs catchup (with examples)
- Trigger rules (beyond the basics)
- XComs, Variables, Connections (when and how)
- Debugging failed tasks, zombie tasks, sensor timeouts

### Exercise Project: "Production-Grade DAG"

Build: Multi-source data pipeline with error handling

Include: Retries, alerts, dynamic task generation

Handle: Backfills, idempotency, data quality checks

Document: Why you made each architectural choice

### Interview Prep:

- "Explain how Airflow's scheduler works"
- "Your DAG is failing with zombie tasks - how do you debug?"
- "How do you handle backfills?"

---

## Week 6: Cloud & AWS Essentials

### Deep Dive:

- S3: Not just storage, but versioning, lifecycle, performance
- Lambda: Serverless patterns, cold starts, limits
- Glue: ETL service, crawlers, data catalog
- IAM: Permissions, roles, policies (understand, don't just copy)
- CloudWatch: Monitoring, logs, alarms

### Exercise Project: "Serverless Data Pipeline"

S3 trigger → Lambda → Process data → Load to RDS/Snowflake

Add: Error handling, logging, CloudWatch alarms

Optimize: Cost and performance

Document: Architecture diagram + trade-off decisions

### Interview Prep:

- "Design a data pipeline on AWS"
- "How would you handle errors in a Lambda function?"
- "Explain S3 consistency model"

---

## Week 7: Snowflake Optimization

### Deep Dive:

- How Snowflake actually stores data (micro-partitions, clustering)
- Virtual warehouses: sizing, auto-suspend, multi-cluster
- Query optimization: clustering keys, materialized views, search optimization
- Cost optimization strategies
- Time travel, fail-safe, cloning
- Snowpipe internals

### Exercise Project: "Optimize a Snowflake Warehouse"

Given: Expensive, slow Snowflake setup

Task: Profile queries, optimize tables, right-size warehouses

Measure: Before/after cost and performance

Deliverable: Optimization report with explanations

### Interview Prep:

- "A Snowflake query is slow - walk me through debugging"
- "How does clustering work in Snowflake?"
- "How would you optimize costs?"

---

## Week 8: Data Quality & Testing

### Deep Dive:

- Why data quality matters (with horror stories)
- dbt tests: schema, data, custom tests
- Great Expectations framework
- Data contracts and expectations
- Monitoring data drift

### Exercise Project: "Build a Data Quality Framework"

Add comprehensive tests to previous projects

Implement: Schema validation, null checks, range checks, freshness checks

Set up: Alerts for data quality issues

Document: Testing strategy and why each test matters

### Interview Prep:

- "How do you ensure data quality?"
- "Give an example of a data quality issue you caught"
- "What tests would you add to this pipeline?"

---

# Phase 3: Interview Ready (Weeks 9-12)

Goal: System design + interview patterns + portfolio polish

## Week 9: System Design for Data Engineers

### Deep Dive:

- How to approach system design questions
- Common patterns: Lambda architecture, Kappa architecture, batch vs streaming
- Scalability considerations
- Trade-offs: consistency vs availability, latency vs throughput

### Exercise Project: "Design 5 Data Systems"

Real interview questions:

- "Design a real-time analytics dashboard"
- "Design a data warehouse for a SaaS company"
- "Design a recommendation system data pipeline"
- "Design a CDC (Change Data Capture) system"
- "Design a data lake architecture"

For each: Draw architecture, explain trade-offs, handle follow-ups

### Interview Prep:

- Practice system design with timer (45 min each)
- Record yourself explaining (helps with articulation)
- Review common patterns

---

## Week 10: Streaming & Real-Time

### Deep Dive:

- Kafka basics: topics, partitions, consumers, producers
- When to use streaming vs batch
- Exactly-once semantics
- Windowing and watermarks

### Exercise Project: "Build a Streaming Pipeline"

Kafka → Spark Streaming → Snowflake

Handle: Late data, exactly-once processing

Or simpler: SQS → Lambda → RDS (if Kafka is too much)

### Interview Prep:

- "When would you use streaming vs batch?"
- "Explain Kafka architecture"
- "How do you handle out-of-order events?"

---

## Week 11: Portfolio & Resume Blitz

### Tasks:

- Polish all projects from weeks 1-10
- Create stellar READMEs for each
- Deploy at least 2 projects (make them accessible)
- Write blog posts explaining your projects
- Update resume with quantified achievements
- Update LinkedIn with projects

Deliverable: Portfolio website with 5-6 strong projects

---

## Week 12: Mock Interviews & Final Prep

### Tasks:

- 10+ mock interviews (use Pramp, Interviewing.io, or friends)
- Review all interview prep from weeks 1-11
- Practice explaining projects out loud
- Prepare behavioral stories (STAR method)
- Apply to 50+ companies

Goal: Start interviewing with confidence

---

# How This Works

Each week you get:

- Monday: Concept drop (videos, articles, docs to study)
- Tuesday-Thursday: Hands-on exercises + coding
- Friday-Saturday: Project work
- Sunday: Interview prep + weekly review + deadline

## Accountability:

Each week has a clear deliverable (project + interview prep doc)

You share progress (GitHub commits, LinkedIn posts)

Miss a deadline = adjust next week, but don't skip content

## Resources I'll provide:

- Curated learning materials (no 100-hour course BS)
- Exercise templates and starter code
- Interview question banks
- Sample solutions with explanations
- Weekly "depth dive" explainers on confusing topics

---

# Real Talk: What This Will Take

## Time commitment:

Realistically: 12-15 hours/week (not 14 hours)

That's ~2 hours weekdays + 4-5 hours weekend

Some weeks will be harder (system design, AWS)

Some will be easier (SQL, Python)

## What you'll struggle with:

- Weeks 1-2: Feeling overwhelmed by depth
- Week 5: Airflow complexity
- Week 9: System design (it's hard for everyone)
- Week 12: Interview anxiety

## What you'll gain:

- Ability to explain WHY, not just WHAT
- Confidence in interviews
- 5-6 portfolio projects that show depth
- Understanding of DE fundamentals
- Interview-specific practice

## Success criteria:

After 12 weeks: You can confidently interview for entry-level DE roles

You can explain your projects in depth

You can tackle system design questions (even if imperfectly)

You have a portfolio that gets you interviews