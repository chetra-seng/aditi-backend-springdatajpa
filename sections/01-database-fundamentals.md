---
layout: center
---
# Database Fundamentals

## Understanding Data Storage

---

# What is a Database?

A database is an organized collection of structured data stored electronically.

<v-clicks>

- **Data persistence** beyond application runtime
- **Structured storage** with relationships
- **Efficient retrieval** and manipulation
- **Concurrent access** by multiple users

</v-clicks>

---

# Real-World Examples

<div class="grid grid-cols-3 gap-4 text-center">
<div>

### Banking
Account balances, transactions

</div>
<div>

### E-commerce
Products, orders, customers

</div>
<div>

### Social Media
Users, posts, connections

</div>
</div>

---

# Types of Databases

| Type | Description | Examples |
|------|-------------|----------|
| **Relational (RDBMS)** | Tables with rows and columns | PostgreSQL, MySQL, Oracle |
| **Document** | JSON/BSON documents | MongoDB, CouchDB |
| **Key-Value** | Simple key-value pairs | Redis, DynamoDB |
| **Graph** | Nodes and relationships | Neo4j, Amazon Neptune |

---

# Why PostgreSQL?

PostgreSQL is the world's most advanced **open-source** relational database.

<v-clicks>

- **Open Source** - Free to use, modify, and distribute
- **ACID Compliant** - Reliable transactions
- **Extensible** - Custom functions, data types, operators
- **Standards Compliant** - Follows SQL standards closely
- **Active Community** - Regular updates and strong support

</v-clicks>

---

# Relational Database Concepts

<v-clicks>

**Tables (Relations)**
- Core structure for storing data
- Organized into rows (records) and columns (fields)

**Schema**
- Logical container for database objects

**Primary Key**
- Unique identifier for each row

**Foreign Key**
- Links tables together
- Enforces referential integrity

</v-clicks>

---

# Visual: Table Structure

```
┌──────────────────────────────────────────────────────┐
│                     students                          │
├────────┬────────────┬───────────┬───────────────────┤
│   id   │ first_name │ last_name │       email       │
├────────┼────────────┼───────────┼───────────────────┤
│   1    │   John     │   Doe     │ john@example.com  │
│   2    │   Jane     │   Smith   │ jane@example.com  │
│   3    │   Bob      │   Wilson  │ bob@example.com   │
└────────┴────────────┴───────────┴───────────────────┘
     ↑         ↑           ↑              ↑
  Primary   Column      Column        Column
    Key
```

---

# The ACID Properties

| Property | Meaning | Example |
|----------|---------|---------|
| **Atomicity** | All or nothing | Transfer completes fully or not at all |
| **Consistency** | Valid state transitions | Account balance never negative |
| **Isolation** | Concurrent transactions don't interfere | Two users don't overwrite each other |
| **Durability** | Committed data persists | Data survives power failure |

---

# SQL Overview

**SQL = Structured Query Language**

<v-clicks>

Four main categories:

- **DDL** (Data Definition Language): `CREATE`, `ALTER`, `DROP`
- **DML** (Data Manipulation Language): `SELECT`, `INSERT`, `UPDATE`, `DELETE`
- **DCL** (Data Control Language): `GRANT`, `REVOKE`
- **TCL** (Transaction Control Language): `BEGIN`, `COMMIT`, `ROLLBACK`

</v-clicks>

---

# SQL Statement Examples

```sql
-- DDL: Define structure
CREATE TABLE students (id SERIAL PRIMARY KEY, name TEXT);

-- DML: Manipulate data
INSERT INTO students (name) VALUES ('John');
SELECT * FROM students;
UPDATE students SET name = 'Jane' WHERE id = 1;
DELETE FROM students WHERE id = 1;

-- TCL: Manage transactions
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```

---

# Key Takeaways

<v-clicks>

- Databases organize and persist structured data
- PostgreSQL is a powerful, open-source RDBMS
- Tables store data in rows and columns
- ACID properties ensure data integrity
- SQL is the language for database operations

</v-clicks>
