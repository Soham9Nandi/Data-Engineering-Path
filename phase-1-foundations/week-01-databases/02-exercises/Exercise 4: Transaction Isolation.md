### Exercise 4: Transaction Isolation

Test isolation levels:

```sql
-- Terminal 1
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM accounts WHERE id = 1;
-- Leave transaction open

-- Terminal 2
UPDATE accounts SET balance = 500 WHERE id = 1;
COMMIT;

-- Back to Terminal 1
SELECT balance FROM accounts WHERE id = 1;
-- What do you see?
COMMIT;

-- Repeat with REPEATABLE READ isolation level
```

**Questions:**
1. What balance did you see in each case?
2. Why the difference?
3. When would you use each isolation level?

---
