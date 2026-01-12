# Data Types & Tables

## PostgreSQL's Type System and Table Design

---

# Numeric Types

| Type | Storage | Range |
|------|---------|-------|
| `SMALLINT` | 2 bytes | -32,768 to 32,767 |
| `INTEGER` | 4 bytes | -2.1B to 2.1B |
| `BIGINT` | 8 bytes | Very large numbers |
| `DECIMAL(p,s)` | variable | Exact precision |
| `REAL` | 4 bytes | 6 decimal digits |
| `DOUBLE PRECISION` | 8 bytes | 15 decimal digits |
| `SERIAL` | 4 bytes | Auto-increment |

---

# Character Types

| Type | Description |
|------|-------------|
| `CHAR(n)` | Fixed-length, padded with spaces |
| `VARCHAR(n)` | Variable-length with limit |
| `TEXT` | Variable unlimited length |

<v-click>

**Best Practice:** Use `TEXT` for most cases; PostgreSQL optimizes it well

```sql
CREATE TABLE users (
    username VARCHAR(50),  -- When you need a max length
    bio TEXT               -- When length varies widely
);
```

</v-click>

---

# Date and Time Types

| Type | Description | Example |
|------|-------------|---------|
| `DATE` | Date only | '2024-01-15' |
| `TIME` | Time only | '14:30:00' |
| `TIMESTAMP` | Date and time | '2024-01-15 14:30:00' |
| `TIMESTAMPTZ` | With timezone | '2024-01-15 14:30:00+07' |

<v-click>

**Best Practice:** Always use `TIMESTAMPTZ` for timestamps

</v-click>

---

# Boolean and UUID

```sql
-- Boolean
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    title TEXT,
    completed BOOLEAN DEFAULT FALSE
);

-- UUID (Universally Unique Identifier)
CREATE TABLE orders (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    customer_name TEXT,
    total DECIMAL(10,2)
);
```

---

# CREATE TABLE Syntax

```sql
CREATE TABLE table_name (
    column1 data_type [constraints],
    column2 data_type [constraints],
    ...
    [table_constraints]
);
```

<v-click>

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

</v-click>

---

# PRIMARY KEY Constraint

Uniquely identifies each row in a table.

```sql
-- Single column
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username TEXT
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

# FOREIGN KEY Constraint

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
```

---

# Foreign Key Actions

```sql
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

| Action | Description |
|--------|-------------|
| `CASCADE` | Delete/update child rows too |
| `SET NULL` | Set FK to NULL |
| `RESTRICT` | Prevent the action |

---

# NOT NULL, DEFAULT, UNIQUE

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,

    -- NOT NULL - value required
    name VARCHAR(100) NOT NULL,

    -- DEFAULT - value if not provided
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    -- UNIQUE - no duplicates
    sku VARCHAR(50) UNIQUE,

    -- CHECK - validate data
    price DECIMAL(10,2) CHECK (price >= 0)
);
```

---

# Modifying Tables

```sql
-- Add column
ALTER TABLE employees ADD COLUMN phone VARCHAR(20);

-- Drop column
ALTER TABLE employees DROP COLUMN phone;

-- Rename column
ALTER TABLE employees RENAME COLUMN name TO full_name;

-- Add constraint
ALTER TABLE employees ADD CONSTRAINT salary_check CHECK (salary > 0);

-- Drop table
DROP TABLE IF EXISTS old_table;
```

---

# Complete Example

```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    birth_date DATE,
    enrollment_date DATE DEFAULT CURRENT_DATE,
    gpa DECIMAL(3,2) CHECK (gpa >= 0 AND gpa <= 4.0),
    department_id INTEGER REFERENCES departments(id),
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

# Lab Exercise: Data Types & Tables

**Tasks (30 minutes):**

<v-clicks>

1. Create a `departments` table with `id` and `name`
2. Create a `students` table with:
   - id (auto-generated)
   - first_name, last_name (required)
   - email (unique)
   - department_id (foreign key)
   - enrollment_date (default to today)
3. Create a `courses` table with `id`, `title`, `credits`
4. Verify your tables with `\dt` and `\d tablename`

</v-clicks>

---

# Module 3 Summary

<v-clicks>

- PostgreSQL has a rich type system
- Use appropriate numeric types for precision
- `TEXT` is preferred for variable-length strings
- Always use `TIMESTAMPTZ` for timestamps
- `PRIMARY KEY` uniquely identifies rows
- `FOREIGN KEY` creates relationships
- `NOT NULL`, `DEFAULT`, `UNIQUE`, `CHECK` validate data

</v-clicks>
