# PostgreSQL Complete Course
## 8-Hour Training Program

---

# Module 1: Database Fundamentals
## Duration: 45 minutes

---

## What is a Database?

A database is an organized collection of structured data stored electronically.

**Key Concepts:**
- Data persistence beyond application runtime
- Structured storage with relationships
- Efficient retrieval and manipulation
- Concurrent access by multiple users

**Real-world examples:** Banking systems, e-commerce catalogs, social media platforms

---

## Types of Databases

| Type | Description | Examples |
|------|-------------|----------|
| Relational (RDBMS) | Tables with rows and columns | PostgreSQL, MySQL, Oracle |
| Document | JSON/BSON documents | MongoDB, CouchDB |
| Key-Value | Simple key-value pairs | Redis, DynamoDB |
| Graph | Nodes and relationships | Neo4j, Amazon Neptune |
| Time-Series | Optimized for time-stamped data | InfluxDB, TimescaleDB |

---

## Why PostgreSQL?

PostgreSQL is the world's most advanced open-source relational database.

**Advantages:**
- **Open Source** – Free to use, modify, and distribute
- **ACID Compliant** – Reliable transactions
- **Extensible** – Custom functions, data types, operators
- **Standards Compliant** – Follows SQL standards closely
- **Advanced Features** – JSON, full-text search, GIS support
- **Active Community** – Regular updates and strong support

---

## Relational Database Concepts

**Tables (Relations)**
- Core structure for storing data
- Organized into rows (records) and columns (fields)

**Schema**
- Logical container for database objects
- Helps organize tables, views, functions

**Primary Key**
- Unique identifier for each row
- Cannot be NULL

**Foreign Key**
- Links tables together
- Enforces referential integrity

---

## The ACID Properties

| Property | Meaning | Example |
|----------|---------|---------|
| **Atomicity** | All or nothing | Transfer completes fully or not at all |
| **Consistency** | Valid state transitions | Account balance never negative |
| **Isolation** | Concurrent transactions don't interfere | Two users don't overwrite each other |
| **Durability** | Committed data persists | Data survives power failure |

---

## SQL Overview

**SQL = Structured Query Language**

Four main categories:

- **DDL** (Data Definition Language): CREATE, ALTER, DROP
- **DML** (Data Manipulation Language): SELECT, INSERT, UPDATE, DELETE
- **DCL** (Data Control Language): GRANT, REVOKE
- **TCL** (Transaction Control Language): BEGIN, COMMIT, ROLLBACK

---

# Module 2: PostgreSQL Installation & Setup
## Duration: 30 minutes

---

## Installing PostgreSQL

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

**macOS (Homebrew):**
```bash
brew install postgresql@16
brew services start postgresql@16
```

**Windows:**
Download installer from postgresql.org

---

## PostgreSQL Architecture

```
┌─────────────────────────────────────────┐
│           Client Applications           │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│            Postmaster Process            │
│         (Main Server Process)            │
└──────────────────┬──────────────────────┘
                   │
    ┌──────────────┼──────────────┐
    ▼              ▼              ▼
┌────────┐   ┌────────┐    ┌────────┐
│Backend │   │Backend │    │Backend │
│Process │   │Process │    │Process │
└────────┘   └────────┘    └────────┘
```

---

## Key Configuration Files

| File | Purpose |
|------|---------|
| `postgresql.conf` | Main server configuration |
| `pg_hba.conf` | Client authentication rules |
| `pg_ident.conf` | User name mapping |

**Location (Linux):** `/etc/postgresql/{version}/main/`

---

## Connecting to PostgreSQL

**Using psql (command line):**
```bash
# Connect as postgres user
sudo -u postgres psql

# Connect to specific database
psql -h localhost -U username -d database_name
```

**Connection string format:**
```
postgresql://user:password@host:port/database
```

---

## psql Essential Commands

| Command | Description |
|---------|-------------|
| `\l` | List all databases |
| `\c dbname` | Connect to database |
| `\dt` | List tables |
| `\d tablename` | Describe table structure |
| `\du` | List users/roles |
| `\q` | Quit psql |
| `\?` | Help for psql commands |
| `\h` | Help for SQL commands |

---

## Creating Your First Database

```sql
-- Create a new database
CREATE DATABASE myapp;

-- Connect to it
\c myapp

-- Create a schema (optional, uses 'public' by default)
CREATE SCHEMA app;

-- Set search path
SET search_path TO app, public;
```

---

# Module 3: Data Types
## Duration: 45 minutes

---

## Numeric Types

| Type | Storage | Range |
|------|---------|-------|
| `SMALLINT` | 2 bytes | -32,768 to 32,767 |
| `INTEGER` | 4 bytes | -2.1B to 2.1B |
| `BIGINT` | 8 bytes | ±9.2 quintillion |
| `DECIMAL(p,s)` | variable | exact precision |
| `NUMERIC(p,s)` | variable | exact precision |
| `REAL` | 4 bytes | 6 decimal digits |
| `DOUBLE PRECISION` | 8 bytes | 15 decimal digits |
| `SERIAL` | 4 bytes | auto-increment |

---

## Character Types

| Type | Description |
|------|-------------|
| `CHAR(n)` | Fixed-length, padded with spaces |
| `VARCHAR(n)` | Variable-length with limit |
| `TEXT` | Variable unlimited length |

**Best Practice:** Use `TEXT` for most cases; PostgreSQL optimizes it well

```sql
CREATE TABLE users (
    username VARCHAR(50),
    bio TEXT
);
```

---

## Date and Time Types

| Type | Description | Example |
|------|-------------|---------|
| `DATE` | Date only | '2024-01-15' |
| `TIME` | Time only | '14:30:00' |
| `TIMESTAMP` | Date and time | '2024-01-15 14:30:00' |
| `TIMESTAMPTZ` | With timezone | '2024-01-15 14:30:00+07' |
| `INTERVAL` | Time span | '2 hours 30 minutes' |

**Best Practice:** Always use `TIMESTAMPTZ` for timestamps

---

## Boolean Type

```sql
-- Valid boolean values
TRUE, FALSE, NULL
't', 'f'
'yes', 'no'
'on', 'off'
'1', '0'

-- Example
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    title TEXT,
    completed BOOLEAN DEFAULT FALSE
);
```

---

## UUID Type

**Universally Unique Identifier** – 128-bit number

```sql
-- Enable extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Use as primary key
CREATE TABLE orders (
    id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
    customer_name TEXT,
    total DECIMAL(10,2)
);

-- Or use built-in function (PostgreSQL 13+)
CREATE TABLE products (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    name TEXT
);
```

---

## JSON Types

| Type | Description |
|------|-------------|
| `JSON` | Stores as text, parsed on each access |
| `JSONB` | Binary format, indexed, faster queries |

```sql
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    data JSONB
);

INSERT INTO events (data) VALUES 
    ('{"type": "click", "page": "/home", "timestamp": "2024-01-15"}');

-- Query JSON
SELECT data->>'type' AS event_type FROM events;
SELECT data->'page' FROM events;  -- Returns JSON
SELECT data->>'page' FROM events; -- Returns text
```

---

## Array Types

```sql
-- Array column
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    tags TEXT[]
);

INSERT INTO products (name, tags) 
VALUES ('Laptop', ARRAY['electronics', 'computers', 'sale']);

-- Query arrays
SELECT * FROM products WHERE 'sale' = ANY(tags);
SELECT * FROM products WHERE tags @> ARRAY['electronics'];

-- Array functions
SELECT array_length(tags, 1) FROM products;
SELECT unnest(tags) FROM products;
```

---

## Special Types

| Type | Use Case |
|------|----------|
| `BYTEA` | Binary data (images, files) |
| `INET` | IPv4 or IPv6 addresses |
| `CIDR` | Network addresses |
| `MACADDR` | MAC addresses |
| `MONEY` | Currency (locale-aware) |
| `POINT, LINE, POLYGON` | Geometric data |
| `TSVECTOR, TSQUERY` | Full-text search |

---

# Module 4: Creating Tables & Constraints
## Duration: 60 minutes

---

## CREATE TABLE Syntax

```sql
CREATE TABLE table_name (
    column1 data_type [constraints],
    column2 data_type [constraints],
    ...
    [table_constraints]
);
```

**Example:**
```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    hire_date DATE DEFAULT CURRENT_DATE,
    salary DECIMAL(10,2) CHECK (salary > 0)
);
```

---

## PRIMARY KEY Constraint

Uniquely identifies each row in a table.

```sql
-- Single column (inline)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username TEXT
);

-- Single column (table-level)
CREATE TABLE users (
    id SERIAL,
    username TEXT,
    PRIMARY KEY (id)
);

-- Composite primary key
CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id)
);
```

---

## FOREIGN KEY Constraint

Creates relationships between tables.

```sql
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department_id INTEGER REFERENCES departments(id)
);

-- With explicit constraint name and actions
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department_id INTEGER,
    CONSTRAINT fk_department 
        FOREIGN KEY (department_id) 
        REFERENCES departments(id)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);
```

---

## Foreign Key Actions

| Action | On DELETE | On UPDATE |
|--------|-----------|-----------|
| `CASCADE` | Delete child rows | Update child FK values |
| `SET NULL` | Set FK to NULL | Set FK to NULL |
| `SET DEFAULT` | Set FK to default | Set FK to default |
| `RESTRICT` | Prevent deletion | Prevent update |
| `NO ACTION` | Same as RESTRICT (default) | Same as RESTRICT |

---

## NOT NULL & DEFAULT

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    
    -- NOT NULL - value required
    name VARCHAR(100) NOT NULL,
    
    -- DEFAULT - value if not provided
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Both combined
    stock INTEGER NOT NULL DEFAULT 0
);
```

---

## UNIQUE Constraint

Ensures no duplicate values in a column.

```sql
-- Single column
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE
);

-- Multiple columns (composite unique)
CREATE TABLE user_roles (
    user_id INTEGER,
    role_id INTEGER,
    UNIQUE (user_id, role_id)
);

-- Named constraint
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    sku VARCHAR(50),
    CONSTRAINT unique_sku UNIQUE (sku)
);
```

---

## CHECK Constraint

Validates data against a condition.

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2) CHECK (price >= 0),
    discount_percent INTEGER CHECK (discount_percent BETWEEN 0 AND 100),
    
    -- Named constraint
    CONSTRAINT valid_quantity CHECK (quantity >= 0),
    
    -- Multi-column check
    CONSTRAINT valid_dates CHECK (end_date > start_date)
);
```

---

## Modifying Tables (ALTER TABLE)

```sql
-- Add column
ALTER TABLE employees ADD COLUMN phone VARCHAR(20);

-- Drop column
ALTER TABLE employees DROP COLUMN phone;

-- Rename column
ALTER TABLE employees RENAME COLUMN name TO full_name;

-- Change data type
ALTER TABLE employees ALTER COLUMN salary TYPE NUMERIC(12,2);

-- Add constraint
ALTER TABLE employees ADD CONSTRAINT salary_positive CHECK (salary > 0);

-- Drop constraint
ALTER TABLE employees DROP CONSTRAINT salary_positive;

-- Rename table
ALTER TABLE employees RENAME TO staff;
```

---

## Dropping Tables

```sql
-- Basic drop (fails if table doesn't exist)
DROP TABLE table_name;

-- Safe drop
DROP TABLE IF EXISTS table_name;

-- Drop with dependencies (cascades to dependent objects)
DROP TABLE table_name CASCADE;

-- Drop multiple tables
DROP TABLE table1, table2, table3;
```

---

# Module 5: Basic CRUD Operations
## Duration: 60 minutes

---

## INSERT - Adding Data

```sql
-- Insert single row
INSERT INTO employees (first_name, last_name, email)
VALUES ('John', 'Doe', 'john@example.com');

-- Insert multiple rows
INSERT INTO employees (first_name, last_name, email)
VALUES 
    ('Jane', 'Smith', 'jane@example.com'),
    ('Bob', 'Wilson', 'bob@example.com'),
    ('Alice', 'Brown', 'alice@example.com');

-- Insert with RETURNING (get inserted data back)
INSERT INTO employees (first_name, last_name, email)
VALUES ('Mike', 'Johnson', 'mike@example.com')
RETURNING id, first_name, last_name;
```

---

## INSERT - Advanced

```sql
-- Insert from SELECT
INSERT INTO employee_archive (id, name, email)
SELECT id, first_name || ' ' || last_name, email
FROM employees
WHERE status = 'inactive';

-- Insert with DEFAULT values
INSERT INTO products (name) VALUES ('Widget');
-- Other columns get their DEFAULT values

-- Upsert (INSERT or UPDATE on conflict)
INSERT INTO products (sku, name, price)
VALUES ('ABC123', 'Widget', 9.99)
ON CONFLICT (sku) 
DO UPDATE SET price = EXCLUDED.price, name = EXCLUDED.name;
```

---

## SELECT - Basic Queries

```sql
-- Select all columns
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

## WHERE Clause - Filtering

```sql
-- Comparison operators
SELECT * FROM products WHERE price > 100;
SELECT * FROM products WHERE price >= 100;
SELECT * FROM products WHERE price < 100;
SELECT * FROM products WHERE price <= 100;
SELECT * FROM products WHERE price = 100;
SELECT * FROM products WHERE price <> 100;  -- not equal
SELECT * FROM products WHERE price != 100;  -- not equal

-- Multiple conditions
SELECT * FROM products WHERE price > 50 AND stock > 0;
SELECT * FROM products WHERE category = 'Electronics' OR category = 'Books';
SELECT * FROM products WHERE NOT (price > 100);
```

---

## WHERE Clause - Advanced Operators

```sql
-- IN operator
SELECT * FROM products WHERE category IN ('Electronics', 'Books', 'Toys');

-- BETWEEN operator
SELECT * FROM orders WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';

-- LIKE operator (pattern matching)
SELECT * FROM products WHERE name LIKE 'A%';      -- starts with A
SELECT * FROM products WHERE name LIKE '%phone%'; -- contains phone
SELECT * FROM products WHERE name LIKE '_____';   -- exactly 5 characters

-- ILIKE (case-insensitive)
SELECT * FROM products WHERE name ILIKE '%phone%';

-- NULL checks
SELECT * FROM employees WHERE manager_id IS NULL;
SELECT * FROM employees WHERE manager_id IS NOT NULL;
```

---

## ORDER BY - Sorting Results

```sql
-- Ascending order (default)
SELECT * FROM products ORDER BY name;
SELECT * FROM products ORDER BY name ASC;

-- Descending order
SELECT * FROM products ORDER BY price DESC;

-- Multiple columns
SELECT * FROM employees 
ORDER BY department_id ASC, salary DESC;

-- Order by expression
SELECT * FROM products ORDER BY price * (1 - discount/100);

-- NULLS FIRST / NULLS LAST
SELECT * FROM employees ORDER BY manager_id NULLS FIRST;
```

---

## UPDATE - Modifying Data

```sql
-- Update single column
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

-- Update from another table
UPDATE products p
SET stock = i.quantity
FROM inventory i
WHERE p.sku = i.sku;
```

---

## DELETE - Removing Data

```sql
-- Delete specific rows
DELETE FROM employees WHERE id = 1;

-- Delete with condition
DELETE FROM products WHERE stock = 0 AND discontinued = true;

-- Delete all rows (careful!)
DELETE FROM logs;

-- Delete with RETURNING
DELETE FROM employees 
WHERE hire_date < '2020-01-01'
RETURNING *;

-- TRUNCATE - faster for deleting all rows
TRUNCATE TABLE logs;
TRUNCATE TABLE logs RESTART IDENTITY;  -- Reset serial sequences
```

---

# Module 6: Joins & Relationships
## Duration: 60 minutes

---

## Understanding Joins

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

## INNER JOIN

Returns only rows with matching values in both tables.

```sql
SELECT 
    e.first_name,
    e.last_name,
    d.name AS department
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

```
employees          departments         Result
┌────┬───────┐    ┌────┬─────────┐    ┌───────┬─────────┐
│ id │ name  │    │ id │ name    │    │ name  │ dept    │
├────┼───────┤    ├────┼─────────┤    ├───────┼─────────┤
│ 1  │ John  │────│ 1  │ Sales   │    │ John  │ Sales   │
│ 2  │ Jane  │────│ 2  │ IT      │    │ Jane  │ IT      │
│ 3  │ Bob   │ X  │ 3  │ HR      │    └───────┴─────────┘
└────┴───────┘    └────┴─────────┘
 (dept_id NULL)
```

---

## LEFT JOIN (LEFT OUTER JOIN)

Returns all rows from the left table, plus matching rows from the right.

```sql
SELECT 
    e.first_name,
    e.last_name,
    d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

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

---

## RIGHT JOIN (RIGHT OUTER JOIN)

Returns all rows from the right table, plus matching rows from the left.

```sql
SELECT 
    e.first_name,
    d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

```
Result:
┌───────┬─────────┐
│ name  │ dept    │
├───────┼─────────┤
│ John  │ Sales   │
│ Jane  │ IT      │
│ NULL  │ HR      │  ◀── Included even without match
└───────┴─────────┘
```

---

## FULL OUTER JOIN

Returns all rows when there's a match in either table.

```sql
SELECT 
    e.first_name,
    d.name AS department
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;
```

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

---

## CROSS JOIN

Returns the Cartesian product (all combinations).

```sql
SELECT 
    p.name AS product,
    c.name AS color
FROM products p
CROSS JOIN colors c;
```

```
products × colors = result
  (3)    ×   (2)   =  (6)

┌─────────┬───────┐
│ product │ color │
├─────────┼───────┤
│ Shirt   │ Red   │
│ Shirt   │ Blue  │
│ Pants   │ Red   │
│ Pants   │ Blue  │
│ Hat     │ Red   │
│ Hat     │ Blue  │
└─────────┴───────┘
```

---

## Self Join

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
    e2.first_name AS employee2,
    e1.department_id
FROM employees e1
JOIN employees e2 ON e1.department_id = e2.department_id
WHERE e1.id < e2.id;  -- Avoid duplicates
```

---

## Multiple Table Joins

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

## Join Best Practices

**1. Use explicit JOIN syntax (not WHERE clause joins)**
```sql
-- Good ✓
SELECT * FROM a JOIN b ON a.id = b.a_id;

-- Avoid ✗
SELECT * FROM a, b WHERE a.id = b.a_id;
```

**2. Always use table aliases**
```sql
SELECT e.name, d.name FROM employees e JOIN departments d ON e.dept_id = d.id;
```

**3. Join on indexed columns for performance**

**4. Consider join order for complex queries**

---

# Module 7: Aggregate Functions & GROUP BY
## Duration: 45 minutes

---

## Aggregate Functions

| Function | Description |
|----------|-------------|
| `COUNT(*)` | Count all rows |
| `COUNT(column)` | Count non-NULL values |
| `COUNT(DISTINCT column)` | Count unique values |
| `SUM(column)` | Total of values |
| `AVG(column)` | Average of values |
| `MIN(column)` | Minimum value |
| `MAX(column)` | Maximum value |
| `ARRAY_AGG(column)` | Collect into array |
| `STRING_AGG(column, delimiter)` | Concatenate strings |

---

## Basic Aggregation Examples

```sql
-- Count all employees
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

## GROUP BY Clause

Groups rows that have the same values.

```sql
-- Count employees per department
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

## GROUP BY with Joins

```sql
-- Sales by department name
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

## HAVING Clause

Filters groups (like WHERE, but for aggregates).

```sql
-- Departments with more than 5 employees
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

## WHERE vs HAVING

| WHERE | HAVING |
|-------|--------|
| Filters rows | Filters groups |
| Before grouping | After grouping |
| Can't use aggregates | Can use aggregates |

```sql
SELECT 
    department_id,
    AVG(salary) AS avg_salary
FROM employees
WHERE status = 'active'      -- Filter rows first
GROUP BY department_id
HAVING AVG(salary) > 50000;  -- Then filter groups
```

---

## Advanced Aggregation

```sql
-- Multiple grouping columns
SELECT 
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    COUNT(*) AS order_count,
    SUM(total) AS revenue
FROM orders
GROUP BY year, month
ORDER BY year, month;

-- String aggregation
SELECT 
    department_id,
    STRING_AGG(first_name, ', ' ORDER BY first_name) AS employees
FROM employees
GROUP BY department_id;

-- Array aggregation
SELECT 
    category,
    ARRAY_AGG(name ORDER BY name) AS products
FROM products
GROUP BY category;
```

---

# Module 8: Subqueries & CTEs
## Duration: 45 minutes

---

## What is a Subquery?

A query nested inside another query.

```sql
-- Subquery in WHERE clause
SELECT * FROM products
WHERE price > (SELECT AVG(price) FROM products);

-- Subquery in FROM clause (derived table)
SELECT avg_prices.category, avg_prices.avg_price
FROM (
    SELECT category, AVG(price) AS avg_price
    FROM products
    GROUP BY category
) AS avg_prices
WHERE avg_prices.avg_price > 100;
```

---

## Scalar Subqueries

Returns a single value.

```sql
-- Get employee with highest salary
SELECT * FROM employees
WHERE salary = (SELECT MAX(salary) FROM employees);

-- Compare to department average
SELECT 
    first_name,
    salary,
    salary - (SELECT AVG(salary) FROM employees) AS diff_from_avg
FROM employees;
```

---

## IN / NOT IN with Subqueries

```sql
-- Products that have been ordered
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

## EXISTS / NOT EXISTS

More efficient than IN for large datasets.

```sql
-- Customers with at least one order
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

## Correlated Subqueries

References columns from the outer query.

```sql
-- Employees earning more than their department average
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

## Common Table Expressions (CTEs)

Named temporary result set using WITH clause.

```sql
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
    ds.avg_salary AS dept_avg,
    ds.emp_count
FROM employees e
JOIN department_stats ds ON e.department_id = ds.department_id
WHERE e.salary > ds.avg_salary;
```

---

## Multiple CTEs

```sql
WITH 
monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', order_date) AS month,
        SUM(total) AS revenue
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
),
avg_monthly AS (
    SELECT AVG(revenue) AS avg_revenue
    FROM monthly_sales
)
SELECT 
    ms.month,
    ms.revenue,
    am.avg_revenue,
    CASE 
        WHEN ms.revenue > am.avg_revenue THEN 'Above Average'
        ELSE 'Below Average'
    END AS performance
FROM monthly_sales ms, avg_monthly am
ORDER BY ms.month;
```

---

## Recursive CTEs

For hierarchical or tree-structured data.

```sql
-- Employee hierarchy
WITH RECURSIVE org_chart AS (
    -- Base case: top-level employees (no manager)
    SELECT id, first_name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: employees with managers
    SELECT e.id, e.first_name, e.manager_id, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level, first_name;
```

---

# Module 9: Window Functions
## Duration: 45 minutes

---

## What are Window Functions?

Perform calculations across a set of rows related to the current row.

```sql
SELECT 
    first_name,
    department_id,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) AS dept_avg
FROM employees;
```

Unlike GROUP BY, window functions don't collapse rows!

---

## Window Function Syntax

```sql
function_name(arg) OVER (
    [PARTITION BY partition_expression]
    [ORDER BY sort_expression]
    [frame_clause]
)
```

**PARTITION BY** - Divides rows into groups
**ORDER BY** - Defines the order within each partition
**Frame** - Defines which rows to include

---

## Ranking Functions

```sql
SELECT 
    first_name,
    department_id,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,
    RANK() OVER (ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;
```

| salary | ROW_NUMBER | RANK | DENSE_RANK |
|--------|------------|------|------------|
| 100000 | 1 | 1 | 1 |
| 100000 | 2 | 1 | 1 |
| 90000 | 3 | 3 | 2 |
| 80000 | 4 | 4 | 3 |

---

## Ranking Within Partitions

```sql
-- Top 3 earners per department
SELECT * FROM (
    SELECT 
        first_name,
        department_id,
        salary,
        RANK() OVER (
            PARTITION BY department_id 
            ORDER BY salary DESC
        ) AS rank_in_dept
    FROM employees
) ranked
WHERE rank_in_dept <= 3;
```

---

## LAG and LEAD

Access values from previous or next rows.

```sql
SELECT 
    order_date,
    total,
    LAG(total) OVER (ORDER BY order_date) AS prev_total,
    LEAD(total) OVER (ORDER BY order_date) AS next_total,
    total - LAG(total) OVER (ORDER BY order_date) AS change
FROM orders;
```

| order_date | total | prev_total | next_total | change |
|------------|-------|------------|------------|--------|
| 2024-01-01 | 100 | NULL | 150 | NULL |
| 2024-01-02 | 150 | 100 | 200 | 50 |
| 2024-01-03 | 200 | 150 | 175 | 50 |

---

## FIRST_VALUE and LAST_VALUE

```sql
SELECT 
    first_name,
    department_id,
    salary,
    FIRST_VALUE(first_name) OVER (
        PARTITION BY department_id 
        ORDER BY salary DESC
    ) AS highest_paid,
    LAST_VALUE(first_name) OVER (
        PARTITION BY department_id 
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_paid
FROM employees;
```

---

## Running Totals and Averages

```sql
SELECT 
    order_date,
    total,
    SUM(total) OVER (ORDER BY order_date) AS running_total,
    AVG(total) OVER (ORDER BY order_date) AS running_avg,
    SUM(total) OVER (
        ORDER BY order_date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_3day_sum
FROM orders;
```

---

## Percent and Cumulative Distribution

```sql
SELECT 
    first_name,
    salary,
    PERCENT_RANK() OVER (ORDER BY salary) AS percent_rank,
    CUME_DIST() OVER (ORDER BY salary) AS cumulative_dist,
    NTILE(4) OVER (ORDER BY salary) AS quartile
FROM employees;
```

---

# Module 10: Indexes & Performance
## Duration: 45 minutes

---

## Why Indexes Matter

Without index: Full table scan (O(n))
With index: B-tree lookup (O(log n))

```
Table: 1,000,000 rows
Without index: ~1,000,000 operations
With B-tree index: ~20 operations
```

---

## Creating Indexes

```sql
-- Basic index
CREATE INDEX idx_employees_email ON employees(email);

-- Unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Multi-column (composite) index
CREATE INDEX idx_orders_customer_date 
ON orders(customer_id, order_date);

-- Partial index (only some rows)
CREATE INDEX idx_orders_pending 
ON orders(status) 
WHERE status = 'pending';

-- Expression index
CREATE INDEX idx_users_lower_email 
ON users(LOWER(email));
```

---

## Index Types

| Type | Use Case |
|------|----------|
| **B-tree** (default) | Equality, range, sorting |
| **Hash** | Equality only |
| **GiST** | Geometric, full-text |
| **GIN** | Arrays, JSON, full-text |
| **BRIN** | Large tables with natural order |

```sql
-- GIN for JSONB
CREATE INDEX idx_events_data ON events USING GIN(data);

-- GIN for arrays
CREATE INDEX idx_products_tags ON products USING GIN(tags);
```

---

## EXPLAIN - Understanding Query Plans

```sql
-- Basic explain
EXPLAIN SELECT * FROM employees WHERE email = 'john@example.com';

-- With execution stats
EXPLAIN ANALYZE SELECT * FROM employees WHERE email = 'john@example.com';

-- Verbose output
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) 
SELECT * FROM employees WHERE department_id = 1;
```

---

## Reading EXPLAIN Output

```
Seq Scan on employees  (cost=0.00..155.00 rows=5000 width=72)
  Filter: (department_id = 1)

Index Scan using idx_emp_dept on employees (cost=0.29..8.31 rows=1 width=72)
  Index Cond: (department_id = 1)
```

Key metrics:
- **Cost**: Estimated effort (startup..total)
- **Rows**: Estimated row count
- **Width**: Average row size in bytes
- **Actual time**: Real execution time (with ANALYZE)

---

## Scan Types

| Scan Type | Description | When Used |
|-----------|-------------|-----------|
| Seq Scan | Reads entire table | Small tables, no index |
| Index Scan | Uses index + heap | Selective queries |
| Index Only Scan | Uses index only | Query needs only indexed columns |
| Bitmap Scan | Combines multiple indexes | OR conditions |

---

## Index Best Practices

**DO:**
- Index foreign keys
- Index columns in WHERE clauses
- Index columns used in ORDER BY
- Use partial indexes for filtered queries
- Consider covering indexes for read-heavy queries

**DON'T:**
- Index every column
- Index tiny tables
- Create indexes before understanding queries
- Forget indexes slow down writes

---

## Maintaining Indexes

```sql
-- List indexes
SELECT indexname, indexdef 
FROM pg_indexes 
WHERE tablename = 'employees';

-- Check index usage
SELECT 
    indexrelname AS index_name,
    idx_scan AS times_used,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE schemaname = 'public';

-- Remove unused index
DROP INDEX idx_unused;

-- Rebuild index
REINDEX INDEX idx_employees_email;
```

---

# Module 11: Transactions & Concurrency
## Duration: 30 minutes

---

## What is a Transaction?

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

## Transaction Properties (ACID)

```
BEGIN TRANSACTION
├── Operation 1 ─┐
├── Operation 2  ├─── Atomicity: All succeed or all fail
├── Operation 3 ─┘
│
├── Consistency: Database stays valid
├── Isolation: Other transactions don't see partial changes
└── Durability: Once committed, data persists
COMMIT
```

---

## SAVEPOINT - Partial Rollback

```sql
BEGIN;

INSERT INTO orders (customer_id, total) VALUES (1, 100);
SAVEPOINT order_created;

INSERT INTO order_items (order_id, product_id) VALUES (1, 1);
-- Oops, wrong product
ROLLBACK TO SAVEPOINT order_created;

INSERT INTO order_items (order_id, product_id) VALUES (1, 2);

COMMIT;  -- Order is saved, with correct item
```

---

## Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|------------|---------------------|--------------|
| Read Uncommitted | Yes | Yes | Yes |
| Read Committed (default) | No | Yes | Yes |
| Repeatable Read | No | No | Yes* |
| Serializable | No | No | No |

*PostgreSQL prevents phantoms in Repeatable Read

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

---

## Common Concurrency Issues

**Lost Update**
```
T1: Read balance = 100
T2: Read balance = 100
T1: Write balance = 150  (100 + 50)
T2: Write balance = 130  (100 + 30)  -- T1's update lost!
```

**Solution: Use SELECT FOR UPDATE**
```sql
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- Other transactions must wait
UPDATE accounts SET balance = balance + 50 WHERE id = 1;
COMMIT;
```

---

## Locking

```sql
-- Row-level lock
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- Skip locked rows (useful for job queues)
SELECT * FROM tasks WHERE status = 'pending' 
FOR UPDATE SKIP LOCKED LIMIT 1;

-- Advisory locks (application-level)
SELECT pg_advisory_lock(123);
-- Do exclusive work
SELECT pg_advisory_unlock(123);
```

---

# Module 12: Views & Materialized Views
## Duration: 30 minutes

---

## What is a View?

A saved query that acts like a virtual table.

```sql
CREATE VIEW active_employees AS
SELECT id, first_name, last_name, email, department_id
FROM employees
WHERE status = 'active';

-- Use it like a table
SELECT * FROM active_employees WHERE department_id = 1;
```

---

## Benefits of Views

- **Simplicity**: Hide complex queries
- **Security**: Restrict access to specific columns/rows
- **Abstraction**: Change underlying tables without breaking apps
- **Consistency**: Ensure everyone uses the same logic

```sql
-- Security: Hide salary from certain users
CREATE VIEW employee_public AS
SELECT id, first_name, last_name, email, department_id
FROM employees;  -- No salary!

GRANT SELECT ON employee_public TO readonly_user;
```

---

## Updatable Views

Simple views can be updated.

```sql
CREATE VIEW marketing_employees AS
SELECT * FROM employees WHERE department_id = 3;

-- This works if the view is simple enough
INSERT INTO marketing_employees (first_name, last_name, department_id)
VALUES ('New', 'Person', 3);

-- WITH CHECK OPTION prevents violations
CREATE VIEW marketing_employees AS
SELECT * FROM employees WHERE department_id = 3
WITH CHECK OPTION;

-- This would fail (department_id != 3)
INSERT INTO marketing_employees (first_name, last_name, department_id)
VALUES ('New', 'Person', 5);  -- Error!
```

---

## Materialized Views

Stores query results physically. Great for expensive queries.

```sql
-- Create materialized view
CREATE MATERIALIZED VIEW monthly_sales_summary AS
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    COUNT(*) AS order_count,
    SUM(total) AS revenue
FROM orders
GROUP BY DATE_TRUNC('month', order_date);

-- Query it (fast!)
SELECT * FROM monthly_sales_summary;

-- Refresh the data
REFRESH MATERIALIZED VIEW monthly_sales_summary;

-- Refresh without locking
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_sales_summary;
-- Requires a unique index
```

---

## View vs Materialized View

| Aspect | View | Materialized View |
|--------|------|-------------------|
| Storage | None (query only) | Stores data |
| Speed | Runs query each time | Fast (pre-computed) |
| Freshness | Always current | Stale until refreshed |
| Indexes | No | Yes |
| Use case | Simple abstraction | Expensive queries, reports |

---

# Module 13: Stored Procedures & Functions
## Duration: 45 minutes

---

## Functions vs Procedures

| Feature | Function | Procedure |
|---------|----------|-----------|
| Returns value | Yes (required) | Optional (OUT params) |
| Can use in SELECT | Yes | No |
| Transaction control | No | Yes (COMMIT/ROLLBACK) |
| Called with | SELECT func() | CALL proc() |

---

## Creating Functions

```sql
-- Simple function
CREATE FUNCTION add_numbers(a INTEGER, b INTEGER)
RETURNS INTEGER
LANGUAGE SQL
AS $$
    SELECT a + b;
$$;

SELECT add_numbers(5, 3);  -- Returns 8

-- Function with PL/pgSQL
CREATE FUNCTION get_employee_name(emp_id INTEGER)
RETURNS TEXT
LANGUAGE plpgsql
AS $$
DECLARE
    emp_name TEXT;
BEGIN
    SELECT first_name || ' ' || last_name INTO emp_name
    FROM employees WHERE id = emp_id;
    RETURN emp_name;
END;
$$;
```

---

## Functions with Control Flow

```sql
CREATE FUNCTION calculate_bonus(
    salary NUMERIC,
    performance_rating INTEGER
)
RETURNS NUMERIC
LANGUAGE plpgsql
AS $$
BEGIN
    IF performance_rating >= 5 THEN
        RETURN salary * 0.20;
    ELSIF performance_rating >= 3 THEN
        RETURN salary * 0.10;
    ELSE
        RETURN salary * 0.05;
    END IF;
END;
$$;

SELECT first_name, salary, 
       calculate_bonus(salary, performance_rating) AS bonus
FROM employees;
```

---

## Functions Returning Tables

```sql
CREATE FUNCTION get_department_employees(dept_id INTEGER)
RETURNS TABLE (
    employee_id INTEGER,
    full_name TEXT,
    salary NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT id, first_name || ' ' || last_name, employees.salary
    FROM employees
    WHERE department_id = dept_id;
END;
$$;

-- Use it
SELECT * FROM get_department_employees(1);
```

---

## Stored Procedures

```sql
CREATE PROCEDURE transfer_funds(
    from_account INTEGER,
    to_account INTEGER,
    amount NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Deduct from source
    UPDATE accounts SET balance = balance - amount
    WHERE id = from_account;
    
    -- Add to destination
    UPDATE accounts SET balance = balance + amount
    WHERE id = to_account;
    
    -- Commit the transaction
    COMMIT;
END;
$$;

-- Call the procedure
CALL transfer_funds(1, 2, 100.00);
```

---

## Error Handling

```sql
CREATE FUNCTION safe_divide(a NUMERIC, b NUMERIC)
RETURNS NUMERIC
LANGUAGE plpgsql
AS $$
BEGIN
    IF b = 0 THEN
        RAISE EXCEPTION 'Division by zero';
    END IF;
    RETURN a / b;
EXCEPTION
    WHEN division_by_zero THEN
        RAISE NOTICE 'Caught division by zero';
        RETURN NULL;
    WHEN OTHERS THEN
        RAISE NOTICE 'Unknown error: %', SQLERRM;
        RETURN NULL;
END;
$$;
```

---

# Module 14: Triggers
## Duration: 30 minutes

---

## What is a Trigger?

Automatically executes a function when a table event occurs.

```
┌─────────────────────────────────────────┐
│           INSERT / UPDATE / DELETE       │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────┐
│   BEFORE trigger → Modify or cancel      │
└──────────────────┬───────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────┐
│   Actual data modification               │
└──────────────────┬───────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────┐
│   AFTER trigger → React to changes       │
└──────────────────────────────────────────┘
```

---

## Creating a Trigger Function

```sql
-- Step 1: Create trigger function
CREATE FUNCTION update_modified_timestamp()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$;

-- Step 2: Attach trigger to table
CREATE TRIGGER set_updated_at
    BEFORE UPDATE ON employees
    FOR EACH ROW
    EXECUTE FUNCTION update_modified_timestamp();
```

---

## Trigger Types

```sql
-- BEFORE trigger (can modify data)
CREATE TRIGGER validate_salary
    BEFORE INSERT OR UPDATE ON employees
    FOR EACH ROW
    EXECUTE FUNCTION check_salary_range();

-- AFTER trigger (for reactions/logging)
CREATE TRIGGER log_salary_change
    AFTER UPDATE OF salary ON employees
    FOR EACH ROW
    EXECUTE FUNCTION log_salary_audit();

-- INSTEAD OF (for views)
CREATE TRIGGER view_insert
    INSTEAD OF INSERT ON employee_view
    FOR EACH ROW
    EXECUTE FUNCTION handle_view_insert();
```

---

## Practical Trigger Examples

**Audit Trail:**
```sql
CREATE FUNCTION audit_employee_changes()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO employee_audit (
        employee_id, action, old_data, new_data, changed_at
    ) VALUES (
        COALESCE(NEW.id, OLD.id),
        TG_OP,
        row_to_json(OLD),
        row_to_json(NEW),
        CURRENT_TIMESTAMP
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER employee_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON employees
    FOR EACH ROW EXECUTE FUNCTION audit_employee_changes();
```

---

## OLD vs NEW in Triggers

| Operation | OLD | NEW |
|-----------|-----|-----|
| INSERT | NULL | New row values |
| UPDATE | Original values | Modified values |
| DELETE | Deleted row | NULL |

```sql
CREATE FUNCTION prevent_salary_decrease()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.salary < OLD.salary THEN
        RAISE EXCEPTION 'Salary cannot be decreased';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

---

# Module 15: JSON & Advanced Features
## Duration: 30 minutes

---

## JSONB Operations

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    attributes JSONB
);

INSERT INTO products (name, attributes) VALUES
('Laptop', '{"brand": "Dell", "specs": {"ram": 16, "storage": 512}}');

-- Access operators
SELECT attributes->>'brand' FROM products;           -- Returns text
SELECT attributes->'specs'->>'ram' FROM products;    -- Nested access
SELECT attributes #>> '{specs,ram}' FROM products;   -- Path access
```

---

## JSONB Query Examples

```sql
-- Filter by JSON value
SELECT * FROM products 
WHERE attributes->>'brand' = 'Dell';

-- Filter by nested value
SELECT * FROM products 
WHERE (attributes->'specs'->>'ram')::int >= 16;

-- Check if key exists
SELECT * FROM products 
WHERE attributes ? 'brand';

-- Check if any key exists
SELECT * FROM products 
WHERE attributes ?| array['brand', 'color'];

-- Contains operator
SELECT * FROM products 
WHERE attributes @> '{"brand": "Dell"}';
```

---

## Modifying JSONB

```sql
-- Set/update a value
UPDATE products 
SET attributes = jsonb_set(attributes, '{specs,ram}', '32')
WHERE id = 1;

-- Add new key
UPDATE products 
SET attributes = attributes || '{"color": "silver"}'
WHERE id = 1;

-- Remove key
UPDATE products 
SET attributes = attributes - 'color'
WHERE id = 1;

-- Remove nested key
UPDATE products 
SET attributes = attributes #- '{specs,storage}'
WHERE id = 1;
```

---

## Full-Text Search

```sql
-- Create text search column
ALTER TABLE articles ADD COLUMN search_vector TSVECTOR;

-- Update with document content
UPDATE articles SET search_vector = 
    to_tsvector('english', title || ' ' || content);

-- Create GIN index
CREATE INDEX idx_articles_search ON articles USING GIN(search_vector);

-- Search
SELECT * FROM articles 
WHERE search_vector @@ to_tsquery('english', 'postgresql & tutorial');

-- With ranking
SELECT title, ts_rank(search_vector, query) AS rank
FROM articles, to_tsquery('english', 'database') query
WHERE search_vector @@ query
ORDER BY rank DESC;
```

---

## Useful PostgreSQL Extensions

```sql
-- UUID generation
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Fuzzy string matching
CREATE EXTENSION IF NOT EXISTS pg_trgm;
SELECT * FROM users WHERE name % 'Jon';  -- Finds "John", "Jon"

-- Cryptographic functions
CREATE EXTENSION IF NOT EXISTS pgcrypto;
SELECT crypt('password', gen_salt('bf'));

-- Geographic data
CREATE EXTENSION IF NOT EXISTS postgis;
```

---

# Module 16: Backup & Security
## Duration: 30 minutes

---

## Backup Strategies

| Method | Use Case | Command |
|--------|----------|---------|
| pg_dump | Single database | `pg_dump dbname > backup.sql` |
| pg_dumpall | All databases | `pg_dumpall > all_backup.sql` |
| pg_basebackup | Physical backup | `pg_basebackup -D /backup` |
| COPY | Single table export | `COPY table TO '/path/file.csv'` |

---

## pg_dump Examples

```bash
# SQL format (human readable)
pg_dump -U postgres mydb > backup.sql

# Custom format (compressed, flexible restore)
pg_dump -U postgres -Fc mydb > backup.dump

# Directory format (parallel backup)
pg_dump -U postgres -Fd -j 4 mydb -f backup_dir/

# Specific tables only
pg_dump -U postgres -t employees -t departments mydb > partial.sql

# Schema only (no data)
pg_dump -U postgres --schema-only mydb > schema.sql

# Data only (no schema)
pg_dump -U postgres --data-only mydb > data.sql
```

---

## Restoring Backups

```bash
# Restore SQL format
psql -U postgres mydb < backup.sql

# Restore custom format
pg_restore -U postgres -d mydb backup.dump

# Restore with options
pg_restore -U postgres -d mydb --clean --if-exists backup.dump

# Restore specific tables
pg_restore -U postgres -d mydb -t employees backup.dump

# Parallel restore
pg_restore -U postgres -d mydb -j 4 backup_dir/
```

---

## User & Role Management

```sql
-- Create a user (login role)
CREATE USER app_user WITH PASSWORD 'secure_password';

-- Create a role (for grouping permissions)
CREATE ROLE readonly;

-- Grant privileges
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
GRANT readonly TO app_user;

-- Revoke privileges
REVOKE INSERT, UPDATE, DELETE ON employees FROM app_user;

-- Alter user
ALTER USER app_user WITH PASSWORD 'new_password';

-- Drop user
DROP USER app_user;
```

---

## Row-Level Security (RLS)

```sql
-- Enable RLS on table
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Create policy: users see only their own documents
CREATE POLICY user_documents ON documents
    FOR ALL
    USING (owner_id = current_user_id());

-- Create policy for SELECT only
CREATE POLICY read_public_docs ON documents
    FOR SELECT
    USING (is_public = true);

-- Admin bypass
CREATE POLICY admin_all ON documents
    FOR ALL
    TO admin_role
    USING (true);
```

---

## Security Best Practices

1. **Use strong passwords** and consider password policies
2. **Limit superuser access** - create specific roles
3. **Use SSL/TLS** for connections
4. **Configure pg_hba.conf** carefully
5. **Regular backups** with tested restores
6. **Keep PostgreSQL updated**
7. **Use Row-Level Security** for multi-tenant apps
8. **Audit sensitive operations**

---

# Module 17: Best Practices Summary
## Duration: 15 minutes

---

## Naming Conventions

```sql
-- Tables: lowercase, plural, snake_case
CREATE TABLE user_accounts (...);
CREATE TABLE order_items (...);

-- Columns: lowercase, snake_case
first_name, created_at, is_active

-- Indexes: idx_tablename_columns
CREATE INDEX idx_users_email ON users(email);

-- Foreign keys: fk_tablename_reference
CONSTRAINT fk_orders_customer FOREIGN KEY (customer_id)...

-- Primary keys: table_id or just id
id, user_id
```

---

## Query Best Practices

```sql
-- Use explicit column names (not SELECT *)
SELECT id, name, email FROM users;

-- Use table aliases in joins
SELECT u.name, o.total
FROM users u
JOIN orders o ON u.id = o.user_id;

-- Avoid N+1 queries - use joins or batch fetching
-- Bad: Loop with individual queries
-- Good: Single query with JOIN

-- Use EXPLAIN ANALYZE for slow queries
EXPLAIN ANALYZE SELECT ...;

-- Use prepared statements (prevent SQL injection)
PREPARE get_user (int) AS SELECT * FROM users WHERE id = $1;
EXECUTE get_user(1);
```

---

## Schema Design Tips

- **Normalize appropriately** (usually 3NF is good)
- **Use appropriate data types** (don't VARCHAR everything)
- **Add constraints** (NOT NULL, CHECK, FK)
- **Index foreign keys** and frequently queried columns
- **Use TIMESTAMPTZ** for timestamps
- **Consider partitioning** for very large tables
- **Document your schema** with COMMENT

```sql
COMMENT ON TABLE users IS 'Stores user account information';
COMMENT ON COLUMN users.status IS 'active, suspended, or deleted';
```

---

## Common Mistakes to Avoid

❌ Using SELECT * in production code
❌ Missing indexes on foreign keys
❌ Not using transactions for related operations
❌ Storing sensitive data in plain text
❌ Ignoring database backups
❌ Not testing restore procedures
❌ Over-indexing or under-indexing
❌ Using ORM exclusively without understanding SQL

---

# Quick Reference Card

## Essential Commands

```sql
-- Database operations
CREATE DATABASE name;
DROP DATABASE name;
\c database_name

-- Table operations
CREATE TABLE name (...);
ALTER TABLE name ADD/DROP/ALTER ...;
DROP TABLE name;

-- Data operations
INSERT INTO table VALUES (...);
SELECT columns FROM table WHERE condition;
UPDATE table SET column = value WHERE condition;
DELETE FROM table WHERE condition;

-- Transactions
BEGIN; COMMIT; ROLLBACK;

-- Information
\dt          -- List tables
\d table     -- Describe table
\di          -- List indexes
```

---

# Thank You!

## Course Complete 🎉

**Key takeaways:**
- PostgreSQL is powerful, feature-rich, and free
- Master the fundamentals: tables, queries, joins
- Use indexes wisely for performance
- Understand transactions for data integrity
- Practice regularly with real projects

**Resources:**
- Official Documentation: postgresql.org/docs
- PostgreSQL Wiki: wiki.postgresql.org
- Practice: pgexercises.com

---

# Questions?

Contact information and additional resources can be added here.

---

*PostgreSQL Course - 8 Hours*
*Last updated: 2024*
