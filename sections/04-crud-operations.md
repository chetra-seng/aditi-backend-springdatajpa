# CRUD Operations

## Create, Read, Update, Delete

---

# INSERT - Adding Data

```sql
-- DML: Insert single row
INSERT INTO employees (first_name, last_name, email)
VALUES ('John', 'Doe', 'john@example.com');

-- DML: Insert multiple rows
INSERT INTO employees (first_name, last_name, email)
VALUES
    ('Jane', 'Smith', 'jane@example.com'),
    ('Bob', 'Wilson', 'bob@example.com'),
    ('Alice', 'Brown', 'alice@example.com');

-- DML: Insert with RETURNING (get inserted data back)
INSERT INTO employees (first_name, last_name, email)
VALUES ('Mike', 'Johnson', 'mike@example.com')
RETURNING id, first_name, last_name;
```

---

# INSERT - Advanced

```sql
-- DML: Insert from SELECT
INSERT INTO employee_archive (id, name, email)
SELECT id, first_name || ' ' || last_name, email
FROM employees
WHERE status = 'inactive';

-- DML: Upsert (INSERT or UPDATE on conflict)
INSERT INTO products (sku, name, price)
VALUES ('ABC123', 'Widget', 9.99)
ON CONFLICT (sku)
DO UPDATE SET price = EXCLUDED.price, name = EXCLUDED.name;
```

---

# SELECT - Basic Queries

```sql
-- DML: Select all columns
SELECT * FROM employees;

-- Select specific columns
SELECT first_name, last_name, email FROM employees;

-- Column aliases
SELECT
    first_name AS "First Name",
    last_name AS "Last Name",
    salary * 12 AS annual_salary
FROM employees;

-- Distinct values
SELECT DISTINCT department_id FROM employees;

-- Limit results
SELECT * FROM employees LIMIT 10;
SELECT * FROM employees LIMIT 10 OFFSET 20;
```

---

# WHERE Clause - Filtering

```sql
-- DML: Comparison operators
SELECT * FROM products WHERE price > 100;
SELECT * FROM products WHERE price >= 100;
SELECT * FROM products WHERE price < 100;
SELECT * FROM products WHERE price <> 100;  -- not equal

-- Multiple conditions
SELECT * FROM products
WHERE price > 50 AND stock > 0;

SELECT * FROM products
WHERE category = 'Electronics' OR category = 'Books';

SELECT * FROM products
WHERE NOT (price > 100);
```

---

# WHERE - Advanced Operators

```sql
-- DML: IN operator
SELECT * FROM products
WHERE category IN ('Electronics', 'Books', 'Toys');

-- BETWEEN operator
SELECT * FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';

-- LIKE operator (pattern matching)
SELECT * FROM products WHERE name LIKE 'A%';      -- starts with A
SELECT * FROM products WHERE name LIKE '%phone%'; -- contains phone
SELECT * FROM products WHERE name LIKE '_____';   -- exactly 5 chars

-- ILIKE (case-insensitive)
SELECT * FROM products WHERE name ILIKE '%phone%';

-- NULL checks
SELECT * FROM employees WHERE manager_id IS NULL;
SELECT * FROM employees WHERE manager_id IS NOT NULL;
```

---

# ORDER BY - Sorting

```sql
-- DML: Ascending order (default)
SELECT * FROM products ORDER BY name;
SELECT * FROM products ORDER BY name ASC;

-- Descending order
SELECT * FROM products ORDER BY price DESC;

-- Multiple columns
SELECT * FROM employees
ORDER BY department_id ASC, salary DESC;

-- NULLS FIRST / NULLS LAST
SELECT * FROM employees ORDER BY manager_id NULLS FIRST;
```

---

# UPDATE - Modifying Data

```sql
-- DML: Update single column
UPDATE employees SET salary = 50000 WHERE id = 1;

-- Update multiple columns
UPDATE employees
SET salary = 55000, department_id = 3
WHERE id = 1;

-- Update with expression
UPDATE products SET price = price * 1.1;  -- 10% increase

-- Update with RETURNING
UPDATE employees
SET salary = salary * 1.05
WHERE performance_rating >= 4
RETURNING id, first_name, salary;
```

---

# DELETE - Removing Data

```sql
-- DML: Delete specific rows
DELETE FROM employees WHERE id = 1;

-- Delete with condition
DELETE FROM products
WHERE stock = 0 AND discontinued = true;

-- Delete all rows (careful!)
DELETE FROM logs;

-- Delete with RETURNING
DELETE FROM employees
WHERE hire_date < '2020-01-01'
RETURNING *;

-- TRUNCATE - faster for deleting all rows
TRUNCATE TABLE logs;
```

---

# Common Query Patterns

```sql
-- DML: String concatenation
SELECT first_name || ' ' || last_name AS full_name
FROM employees;

-- COALESCE for NULL handling
SELECT COALESCE(middle_name, '') AS middle_name
FROM users;

-- CASE expression
SELECT
    name,
    CASE
        WHEN price < 10 THEN 'Cheap'
        WHEN price < 100 THEN 'Medium'
        ELSE 'Expensive'
    END AS price_category
FROM products;
```

---

# Practice: CRUD

 

<v-clicks>

1. Insert 5 students into your students table
2. Query all students ordered by last name
3. Find students with email containing '@gmail.com'
4. Update a student's email address
5. Delete a student by id
6. Use RETURNING to see affected data

</v-clicks>

---

# Key Takeaways

<v-clicks>

- `INSERT` adds new rows to tables
- `SELECT` retrieves data with filtering and sorting
- `WHERE` filters rows based on conditions
- `ORDER BY` sorts results
- `UPDATE` modifies existing data
- `DELETE` removes rows
- `RETURNING` shows affected data
- Always use `WHERE` with UPDATE/DELETE!

</v-clicks>
