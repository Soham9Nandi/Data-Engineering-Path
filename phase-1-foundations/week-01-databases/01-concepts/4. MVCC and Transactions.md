## 📖 CONCEPT 4: MVCC AND TRANSACTIONS

### ACID Properties

**Atomicity:** All or nothing
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- Both happen or neither
```

**Consistency:** Database stays valid
```sql
-- CHECK constraint ensures balance >= 0
UPDATE accounts SET balance = -100 WHERE id = 1;
-- ERROR: violates check constraint
```

**Isolation:** Transactions don't interfere
```sql
-- Transaction 1
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- Sees 100

-- Transaction 2 (concurrent)
BEGIN;
UPDATE accounts SET balance = 200 WHERE id = 1;
COMMIT;

-- Back to Transaction 1
SELECT balance FROM accounts WHERE id = 1;  -- Still sees 100!
COMMIT;
```

**Durability:** Committed = permanent (WAL)

---

### Isolation Levels

**Read Uncommitted:** (Postgres doesn't support this)
- Can see uncommitted changes
- Dirty reads possible

**Read Committed:** (Postgres default)
- Only sees committed changes
- Each statement sees latest commits

**Repeatable Read:**
- Sees snapshot at transaction start
- Same query returns same results

**Serializable:**
- Strictest level
- Transactions appear to run serially
- May get serialization errors

**Example:**
```sql
-- Session 1
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;  -- Returns 100

-- Session 2
UPDATE accounts SET balance = 200 WHERE id = 1;
COMMIT;

-- Back to Session 1
SELECT balance FROM accounts WHERE id = 1;  -- Still returns 100!
COMMIT;
```

---

### Common Concurrency Issues

**1. Dirty Reads:** Reading uncommitted data
- Prevented in Postgres (minimum Read Committed)

**2. Non-Repeatable Reads:** Row changes between reads
- Prevented with Repeatable Read or Serializable

**3. Phantom Reads:** New rows appear between reads
- Prevented with Repeatable Read or Serializable

**4. Lost Updates:**
```sql
-- Transaction 1
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- 100
-- ... do some calculation ...
UPDATE accounts SET balance = 150 WHERE id = 1;

-- Transaction 2 (concurrent)
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- 100
UPDATE accounts SET balance = 200 WHERE id = 1;  -- Overwrites T1!
COMMIT;

-- Transaction 1
COMMIT;  -- Lost the update from T2!
```

**Solution: SELECT FOR UPDATE**
```sql
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;  -- Locks row
UPDATE accounts SET balance = 150 WHERE id = 1;
COMMIT;
```

---
