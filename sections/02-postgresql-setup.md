---
layout: center
---
# PostgreSQL Setup

## Installation and First Steps

---

# Installing PostgreSQL

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

# PostgreSQL Architecture

```
┌─────────────────────────────────────────┐
│           Client Applications           │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│            Postmaster Process           │
│         (Main Server Process)           │
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

# Connecting to PostgreSQL

**Using psql (command line):**

```bash
# Connect as postgres user
sudo -u postgres psql

# Connect to specific database
psql -h localhost -U username -d database_name

# Connect with connection string
psql postgresql://user:password@host:port/database
```

---

# psql Essential Commands

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

# Creating Your First Database

```sql
-- DDL: Create and manage databases
CREATE DATABASE studentdb;

-- Connect to it
\c studentdb

-- DML: Query database list
SELECT datname FROM pg_database;

-- DDL: Drop database (careful!)
DROP DATABASE IF EXISTS old_database;
```

---

# Database Users

```sql
-- DCL: Create a user
CREATE USER app_user WITH PASSWORD 'secure_password';

-- DCL: Grant privileges
GRANT ALL PRIVILEGES ON DATABASE studentdb TO app_user;

-- List users
\du
```

---

# Practice: Setup

**Tasks (15 minutes):**

<v-clicks>

1. Install PostgreSQL on your machine
2. Start the PostgreSQL service
3. Connect using `psql`
4. Create a database called `studentdb`
5. List all databases with `\l`
6. Connect to your new database with `\c studentdb`

</v-clicks>

---

# Key Takeaways

<v-clicks>

- PostgreSQL can be installed via package managers
- `psql` is the primary command-line interface
- Key commands: `\l`, `\c`, `\dt`, `\d`, `\q`
- Databases are created with `CREATE DATABASE`
- Users are managed with `CREATE USER` and `GRANT`

</v-clicks>
