---
layout: center
---
# Indexes & Transactions

## Performance and Data Integrity

---

# Why Indexes Matter

Without index: Full table scan (O(n))
With index: B-tree lookup (O(log n))

```
Table: 1,000,000 rows
Without index: ~1,000,000 comparisons
With B-tree index: ~20 comparisons
```

---

# Creating Indexes

```sql
-- DDL: Basic index
CREATE INDEX idx_employees_email ON employees(email);

-- DDL: Unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- DDL: Multi-column (composite) index
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);

-- DDL: Drop index
DROP INDEX idx_unused;
```

---

# When to Create Indexes

<v-clicks>

**DO index:**
- Primary keys (automatic)
- Foreign keys
- Columns in WHERE clauses
- Columns in ORDER BY
- Columns in JOIN conditions

**DON'T index:**
- Small tables
- Columns with few unique values
- Columns rarely used in queries

</v-clicks>

---

# EXPLAIN - Understanding Query Plans

```sql
-- DML: Basic explain to see query plan
EXPLAIN SELECT * FROM employees WHERE email = 'john@example.com';

-- DML: With execution stats
EXPLAIN ANALYZE SELECT * FROM employees WHERE email = 'john@example.com';
```

```
-- Without index:
Seq Scan on employees  (cost=0.00..155.00 rows=5000)
  Filter: (email = 'john@example.com')

-- With index:
Index Scan using idx_employees_email  (cost=0.29..8.31 rows=1)
  Index Cond: (email = 'john@example.com')
```

---

# Index Best Practices

<v-clicks>

1. **Index foreign keys** - Essential for join performance
2. **Check query plans** - Use EXPLAIN ANALYZE
3. **Don't over-index** - Indexes slow down writes
4. **Composite indexes** - Column order matters
5. **Maintain indexes** - Rebuild if needed with REINDEX

</v-clicks>

---

# What is a Transaction?

A sequence of operations treated as a single unit of work.

```sql
-- TCL: Start transaction
BEGIN;

-- DML: Modify data within transaction
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- TCL: Save changes
COMMIT;

-- TCL: Or if something went wrong, undo all changes
ROLLBACK;
```

---

# Transaction Example

```sql
-- TCL: Begin transaction
BEGIN;

-- DML: Transfer $100 from account 1 to account 2
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- DML: Check if source has sufficient funds
SELECT balance FROM accounts WHERE id = 1;
-- TCL: If balance < 0
-- ROLLBACK;

-- TCL: Commit transaction
COMMIT;
```

---

# SAVEPOINT - Partial Rollback

```sql
-- TCL: Begin transaction
BEGIN;

-- DML: Insert order
INSERT INTO orders (customer_id, total) VALUES (1, 100);
-- TCL: Create savepoint
SAVEPOINT order_created;

-- DML: Insert order item
INSERT INTO order_items (order_id, product_id) VALUES (1, 1);
-- TCL: Oops, wrong product - rollback to savepoint
ROLLBACK TO SAVEPOINT order_created;

-- DML: Insert correct item
INSERT INTO order_items (order_id, product_id) VALUES (1, 2);

-- TCL: Commit - order is saved with correct item
COMMIT;
```

---

# Transaction Isolation

```sql
-- TCL: Set isolation level
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;  -- Default
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

| Level | Description |
|-------|-------------|
| Read Committed | See committed changes (default) |
| Repeatable Read | Consistent view within transaction |
| Serializable | Full isolation, like sequential |

---

# Row Locking - What and Why?

**Problem:** Two transactions trying to update the same row simultaneously

**Solution:** Lock rows to prevent conflicts

<v-clicks>

**When you need it:**
- Banking: Prevent double-spending
- Inventory: Prevent overselling
- Job Queues: One worker per task

**Lock Types:**
- `FOR UPDATE` - Exclusive lock, blocks other locks
- `FOR SHARE` - Shared lock, allows other reads
- `SKIP LOCKED` - Skip already locked rows
- `NOWAIT` - Fail immediately if locked

</v-clicks>

---

# Row Locking Examples

```sql
-- DML: FOR UPDATE - Exclusive lock
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- Other transactions WAIT until this commits
UPDATE accounts SET balance = balance + 50 WHERE id = 1;
COMMIT;

-- DML: SKIP LOCKED - Don't wait for locked rows
SELECT * FROM tasks
WHERE status = 'pending'
FOR UPDATE SKIP LOCKED
LIMIT 1;
-- Returns first unlocked task (perfect for job queues!)
```

---

# Real-World Scenario: Job Queue

**Multiple Workers Processing Tasks:**

```sql
-- Worker 1 picks up first available task (id=123)
BEGIN;
SELECT * FROM jobs WHERE status = 'pending'
FOR UPDATE SKIP LOCKED LIMIT 1;
-- Returns: id=123 (now LOCKED by Worker 1)

UPDATE jobs SET status = 'processing', worker_id = 1 WHERE id = 123;
-- ... Worker 1 processing job 123 ...
```

```sql
-- Worker 2 runs THE SAME QUERY at the same time
BEGIN;
SELECT * FROM jobs WHERE status = 'pending'
FOR UPDATE SKIP LOCKED LIMIT 1;
-- Returns: id=124 (SKIPS locked id=123, gets next one!)

UPDATE jobs SET status = 'processing', worker_id = 2 WHERE id = 124;
-- ... Worker 2 processing job 124 ...
```

---

# SKIP LOCKED - The Difference

<div class="grid grid-cols-2 gap-4">
<div>

**WITHOUT `SKIP LOCKED`:**
```
Worker 1: Gets job 123, locks it
Worker 2: Tries to get job 123...
          WAITS... WAITS...
          (blocked until Worker 1 done)
```
❌ Only one worker active at a time

</div>
<div>

**WITH `SKIP LOCKED`:**
```
Worker 1: Gets job 123, locks it
Worker 2: Skips job 123 (locked)
          Gets job 124 immediately!
```
✅ Both workers active simultaneously

</div>
</div>

<v-click>

**Key Point:** `SKIP LOCKED` enables parallel processing by skipping over rows that are already locked by other transactions.

</v-click>

---

# Transaction Best Practices

<v-clicks>

1. **Keep transactions short** - Long transactions block others
2. **Don't hold during user input** - Risk of long locks
3. **Handle errors properly** - Rollback on exception
4. **Use appropriate isolation** - Read Committed is usually fine
5. **Lock in consistent order** - Prevents deadlocks

</v-clicks>

---

# PostgreSQL Part 1 Complete!

**What you've learned:**

<v-clicks>

- Database fundamentals and PostgreSQL setup
- Data types, tables, and constraints
- CRUD operations
- Joins and relationships
- Aggregations and subqueries
- Indexes and transactions

</v-clicks>

---

# Ready for Spring Data JPA!

Next: Connecting Java applications to PostgreSQL

<v-clicks>

- JPA and Hibernate fundamentals
- Entity mapping
- Repository pattern
- Custom queries
- Building production-ready APIs

</v-clicks>
