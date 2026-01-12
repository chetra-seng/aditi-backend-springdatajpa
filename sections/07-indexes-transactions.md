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
-- Basic index
CREATE INDEX idx_employees_email ON employees(email);

-- Unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Multi-column (composite) index
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);

-- Drop index
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
-- Basic explain
EXPLAIN SELECT * FROM employees WHERE email = 'john@example.com';

-- With execution stats
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
BEGIN;                                    -- Start transaction

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;                                   -- Save changes

-- Or if something went wrong
ROLLBACK;                                 -- Undo all changes
```

---

# Transaction Example

```sql
BEGIN;

-- Transfer $100 from account 1 to account 2
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Check if source has sufficient funds
SELECT balance FROM accounts WHERE id = 1;
-- If balance < 0
-- ROLLBACK;

COMMIT;
```

---

# SAVEPOINT - Partial Rollback

```sql
BEGIN;

INSERT INTO orders (customer_id, total) VALUES (1, 100);
SAVEPOINT order_created;

INSERT INTO order_items (order_id, product_id) VALUES (1, 1);
-- Oops, wrong product
ROLLBACK TO SAVEPOINT order_created;

INSERT INTO order_items (order_id, product_id) VALUES (1, 2);

COMMIT;  -- Order is saved with correct item
```

---

# Transaction Isolation

```sql
-- Set isolation level
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

# Row Locking

```sql
-- Lock rows for update
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- Other transactions wait for this row
UPDATE accounts SET balance = balance + 50 WHERE id = 1;
COMMIT;

-- Skip locked rows (useful for job queues)
SELECT * FROM tasks
WHERE status = 'pending'
FOR UPDATE SKIP LOCKED
LIMIT 1;
```

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
