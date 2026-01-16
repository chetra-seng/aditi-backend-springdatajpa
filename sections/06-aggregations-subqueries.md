# Aggregations & Subqueries

## Summarizing Data and Advanced Queries

---

# Aggregate Functions

| Function | Description |
|----------|-------------|
| `COUNT(*)` | Count all rows |
| `COUNT(column)` | Count non-NULL values |
| `COUNT(DISTINCT col)` | Count unique values |
| `SUM(column)` | Total of values |
| `AVG(column)` | Average of values |
| `MIN(column)` | Minimum value |
| `MAX(column)` | Maximum value |

---

# Basic Aggregation Examples

```sql
-- DML: Count all employees
SELECT COUNT(*) AS total_employees FROM employees;

-- Count employees with email
SELECT COUNT(email) AS employees_with_email FROM employees;

-- Sum of all salaries
SELECT SUM(salary) AS total_payroll FROM employees;

-- Average salary
SELECT AVG(salary) AS average_salary FROM employees;

-- Salary range
SELECT
    MIN(salary) AS lowest_salary,
    MAX(salary) AS highest_salary
FROM employees;
```

---

# GROUP BY Clause

Groups rows that have the same values.

```sql
-- DML: Count employees per department
SELECT
    department_id,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department_id;

-- Total sales per product
SELECT
    product_id,
    SUM(quantity) AS total_sold,
    SUM(quantity * unit_price) AS total_revenue
FROM order_items
GROUP BY product_id;
```

---

# GROUP BY with Joins

```sql
-- DML: Employee stats by department name
SELECT
    d.name AS department,
    COUNT(*) AS employee_count,
    SUM(e.salary) AS total_salary,
    ROUND(AVG(e.salary), 2) AS avg_salary
FROM employees e
JOIN departments d ON e.department_id = d.id
GROUP BY d.name
ORDER BY total_salary DESC;
```

---

# HAVING Clause

Filters groups (like WHERE, but for aggregates).

```sql
-- DML: Departments with more than 5 employees
SELECT
    department_id,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5;

-- Products with total sales over $10,000
SELECT
    product_id,
    SUM(quantity * unit_price) AS total_sales
FROM order_items
GROUP BY product_id
HAVING SUM(quantity * unit_price) > 10000
ORDER BY total_sales DESC;
```

---

# WHERE vs HAVING

| WHERE | HAVING |
|-------|--------|
| Filters rows | Filters groups |
| Before grouping | After grouping |
| Can't use aggregates | Can use aggregates |

```sql
-- DML: WHERE filters rows before GROUP BY, HAVING filters groups after
SELECT
    department_id,
    AVG(salary) AS avg_salary
FROM employees
WHERE status = 'active'      -- Filter rows first
GROUP BY department_id
HAVING AVG(salary) > 50000;  -- Then filter groups
```

---

# What is a Subquery?

A query nested inside another query.

```sql
-- DML: Subquery in WHERE clause
SELECT * FROM products
WHERE price > (SELECT AVG(price) FROM products);

-- Subquery in FROM clause
SELECT avg_prices.category, avg_prices.avg_price
FROM (
    SELECT category, AVG(price) AS avg_price
    FROM products
    GROUP BY category
) AS avg_prices
WHERE avg_prices.avg_price > 100;
```

---

# Scalar Subqueries

Returns a single value.

```sql
-- DML: Scalar subquery - Get employee with highest salary
SELECT * FROM employees
WHERE salary = (SELECT MAX(salary) FROM employees);

-- Compare to company average
SELECT
    first_name,
    salary,
    salary - (SELECT AVG(salary) FROM employees) AS diff_from_avg
FROM employees;
```

---

# IN / NOT IN with Subqueries

```sql
-- DML: Using IN with subquery
SELECT * FROM products
WHERE id IN (
    SELECT DISTINCT product_id FROM order_items
);

-- Customers who haven't ordered in 2024
SELECT * FROM customers
WHERE id NOT IN (
    SELECT DISTINCT customer_id
    FROM orders
    WHERE EXTRACT(YEAR FROM order_date) = 2024
);
```

---

# EXISTS / NOT EXISTS

More efficient than IN for large datasets.

```sql
-- DML: Using EXISTS for better performance
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);

-- Products never ordered
SELECT * FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items oi WHERE oi.product_id = p.id
);
```

---

# Correlated Subqueries

References columns from the outer query.

```sql
-- DML: Correlated subquery references outer query
SELECT * FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);

-- Latest order for each customer
SELECT * FROM orders o
WHERE order_date = (
    SELECT MAX(order_date)
    FROM orders
    WHERE customer_id = o.customer_id
);
```

---

# Common Table Expressions (CTEs)

Named temporary result set using WITH clause.

```sql
-- DML: Common Table Expression (CTE) for better readability
WITH department_stats AS (
    SELECT
        department_id,
        AVG(salary) AS avg_salary,
        COUNT(*) AS emp_count
    FROM employees
    GROUP BY department_id
)
SELECT
    e.first_name,
    e.salary,
    ds.avg_salary AS dept_avg
FROM employees e
JOIN department_stats ds ON e.department_id = ds.department_id
WHERE e.salary > ds.avg_salary;
```

---

# Multiple CTEs

```sql
-- DML: Multiple CTEs can be chained together
WITH
monthly_sales AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month,
        SUM(total) AS revenue
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
),
avg_monthly AS (
    SELECT AVG(revenue) AS avg_revenue FROM monthly_sales
)
SELECT
    ms.month,
    ms.revenue,
    am.avg_revenue,
    CASE
        WHEN ms.revenue > am.avg_revenue THEN 'Above'
        ELSE 'Below'
    END AS performance
FROM monthly_sales ms, avg_monthly am
ORDER BY ms.month;
```

---

# Practice: Aggregations & Subqueries

 

<v-clicks>

1. Count students per department
2. Find departments with average GPA above 3.0
3. Find students with GPA above the overall average
4. Use EXISTS to find courses with enrollments
5. Create a CTE for student statistics per department

</v-clicks>

---

# Key Takeaways

<v-clicks>

- Aggregate functions summarize data (COUNT, SUM, AVG, etc.)
- `GROUP BY` groups rows for aggregation
- `HAVING` filters groups (not rows)
- Subqueries nest queries inside other queries
- `IN` / `EXISTS` check for membership
- Correlated subqueries reference outer query
- CTEs improve readability with named results

</v-clicks>
