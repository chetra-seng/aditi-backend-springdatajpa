# Joins & Relationships

## Connecting Tables Together

---

# Understanding Joins

Joins combine rows from two or more tables based on a related column.

```
┌─────────────┐         ┌─────────────┐
│  employees  │         │ departments │
├─────────────┤         ├─────────────┤
│ id          │────────▶│ id          │
│ name        │         │ name        │
│ dept_id     │         │ location    │
└─────────────┘         └─────────────┘
```

---

# INNER JOIN

Returns only rows with matching values in both tables.

```sql
SELECT
    e.first_name,
    e.last_name,
    d.name AS department
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

<v-click>

```
employees          departments         Result
┌────┬───────┐    ┌────┬─────────┐    ┌───────┬─────────┐
│ id │ name  │    │ id │ name    │    │ name  │ dept    │
├────┼───────┤    ├────┼─────────┤    ├───────┼─────────┤
│ 1  │ John  │────│ 1  │ Sales   │    │ John  │ Sales   │
│ 2  │ Jane  │────│ 2  │ IT      │    │ Jane  │ IT      │
│ 3  │ Bob   │ X  │ 3  │ HR      │    └───────┴─────────┘
└────┴───────┘    └────┴─────────┘    Bob excluded (no dept)
```

</v-click>

---

# LEFT JOIN

Returns all rows from the left table, plus matching rows from the right.

```sql
SELECT
    e.first_name,
    e.last_name,
    d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

<v-click>

```
Result:
┌───────┬─────────┐
│ name  │ dept    │
├───────┼─────────┤
│ John  │ Sales   │
│ Jane  │ IT      │
│ Bob   │ NULL    │  ◀── Included even without match
└───────┴─────────┘
```

</v-click>

---

# RIGHT JOIN

Returns all rows from the right table, plus matching rows from the left.

```sql
SELECT
    e.first_name,
    d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

<v-click>

```
Result:
┌───────┬─────────┐
│ name  │ dept    │
├───────┼─────────┤
│ John  │ Sales   │
│ Jane  │ IT      │
│ NULL  │ HR      │  ◀── Included even without employee
└───────┴─────────┘
```

</v-click>

---

# FULL OUTER JOIN

Returns all rows when there's a match in either table.

```sql
SELECT
    e.first_name,
    d.name AS department
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;
```

<v-click>

```
Result:
┌───────┬─────────┐
│ name  │ dept    │
├───────┼─────────┤
│ John  │ Sales   │
│ Jane  │ IT      │
│ Bob   │ NULL    │  ◀── No matching department
│ NULL  │ HR      │  ◀── No matching employee
└───────┴─────────┘
```

</v-click>

---

# Join Visualization

```
      INNER JOIN         LEFT JOIN          RIGHT JOIN
     ┌───┬───┐          ┌───┬───┐          ┌───┬───┐
     │   │###│          │###│###│          │###│   │
     │   │###│          │###│###│          │###│   │
     └───┴───┘          └───┴───┘          └───┴───┘
        A B                A B                A B

                    FULL OUTER JOIN
                      ┌───┬───┐
                      │###│###│
                      │###│###│
                      └───┴───┘
                         A B
```

---

# Self Join

A table joined with itself.

```sql
-- Find employees and their managers
SELECT
    e.first_name AS employee,
    m.first_name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- Find employees in the same department
SELECT
    e1.first_name AS employee1,
    e2.first_name AS employee2
FROM employees e1
JOIN employees e2 ON e1.department_id = e2.department_id
WHERE e1.id < e2.id;  -- Avoid duplicates
```

---

# Multiple Table Joins

```sql
SELECT
    o.id AS order_id,
    c.name AS customer,
    p.name AS product,
    oi.quantity,
    oi.unit_price
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE o.order_date >= '2024-01-01';
```

---

# CROSS JOIN

Returns the Cartesian product (all combinations).

```sql
SELECT
    p.name AS product,
    c.name AS color
FROM products p
CROSS JOIN colors c;
```

Use sparingly - can produce very large results!

---

# Join Best Practices

<v-clicks>

**1. Use explicit JOIN syntax (not WHERE clause joins)**
```sql
-- Good
SELECT * FROM a JOIN b ON a.id = b.a_id;

-- Avoid
SELECT * FROM a, b WHERE a.id = b.a_id;
```

**2. Always use table aliases**

**3. Join on indexed columns for performance**

**4. Be explicit about join type (INNER, LEFT, etc.)**

</v-clicks>

---

# Lab Exercise: Joins

**Tasks (30 minutes):**

<v-clicks>

1. Find all students with their department names (INNER JOIN)
2. Find all students, including those without departments (LEFT JOIN)
3. Find departments with no students
4. List students and their enrolled courses
5. Find students and their "mentor" student (self-join)

</v-clicks>

---

# Module 5 Summary

<v-clicks>

- `INNER JOIN` - Only matching rows from both tables
- `LEFT JOIN` - All from left + matching from right
- `RIGHT JOIN` - All from right + matching from left
- `FULL OUTER JOIN` - All rows from both tables
- Self-joins connect a table to itself
- Use explicit JOIN syntax with aliases
- Join on indexed columns for better performance

</v-clicks>
