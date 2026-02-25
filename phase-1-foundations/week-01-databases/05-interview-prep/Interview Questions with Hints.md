## 🎤 INTERVIEW PREP

### Questions You Must Be Able to Answer

**1. Explain how PostgreSQL stores data on disk.**
*Your answer should mention: pages, tuples, MVCC, TOAST*

**2. What is MVCC and why does it matter?**
*Your answer should mention: multiple row versions, no read locks, dead tuples, VACUUM*

**3. Walk me through what happens when you run a SELECT query.**
*Your answer should mention: parser, planner, executor, query plan*

**4. How do you debug a slow query?**
*Your answer should mention: EXPLAIN ANALYZE, check for seq scans, check statistics, add indexes*

**5. When would you NOT add an index?**
*Your answer should mention: small tables, low selectivity, heavy writes*

**6. What's the difference between VACUUM and VACUUM FULL?**
*Your answer should mention: dead tuple cleanup vs table rewrite, locking*

**7. Explain the different types of index scans.**
*Your answer should mention: seq scan, index scan, bitmap heap scan, index-only scan*

**8. What isolation levels does PostgreSQL support?**
*Your answer should mention: Read Committed (default), Repeatable Read, Serializable*

**9. How do you prevent lost updates?**
*Your answer should mention: SELECT FOR UPDATE, optimistic locking, Serializable isolation*

**10. What's a covering index?**
*Your answer should mention: index contains all columns needed, no table lookup, index-only scan*

---