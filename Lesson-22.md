# Lesson 22: Databases in the Software Development Process

> Databases are at the heart of almost every application. As a DevOps engineer, you are not expected to write complex SQL queries like a developer — but you must understand how databases fit into the development lifecycle, how they are configured across environments, and how to manage, replicate, back up, and restore them in production. This lesson covers the full picture.

---

## Table of Contents

1. [Why DevOps Engineers Need to Understand Databases](#1-why-devops-engineers-need-to-understand-databases)
2. [Local vs. Remote Database — Development Strategies](#2-local-vs-remote-database--development-strategies)
3. [Configuring Database Connections in Applications](#3-configuring-database-connections-in-applications)
4. [Database Across Environments](#4-database-across-environments)
5. [Database Types](#5-database-types)
6. [DevOps Responsibilities for Databases](#6-devops-responsibilities-for-databases)
7. [Database Replication](#7-database-replication)
8. [Database Backups](#8-database-backups)
9. [Database Restore & Disaster Recovery](#9-database-restore--disaster-recovery)
10. [Database in CI/CD Pipelines](#10-database-in-cicd-pipelines)
11. [Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)

---

## 1. Why DevOps Engineers Need to Understand Databases

Every application has data — and that data lives in a database. As a DevOps engineer you sit between the development team and the infrastructure. Databases touch both sides:

```
┌──────────────────────────────────────────────────────────────┐
│              DEVOPS & DATABASE INTERSECTION                  │
│                                                              │
│  Developers write app code         DevOps manages            │
│  that talks to the DB              the DB infrastructure     │
│                                                              │
│  Dev side:                         DevOps side:              │
│  • DB connection strings           • Install & configure DB  │
│  • ORM queries                     • Set up replication      │
│  • Schema migrations               • Schedule backups        │
│  • Test data                       • Monitor performance     │
│                                    • Restore on failure      │
│                                    • Manage credentials      │
│                                    • Automate DB deployments │
└──────────────────────────────────────────────────────────────┘
```

| DevOps Database Responsibility | Why It Matters |
|-------------------------------|----------------|
| Configure & setup DB servers | Applications can't run without a working DB |
| Manage credentials securely | Exposed DB passwords are a top security breach cause |
| Set up replication | Ensures high availability — no single point of failure |
| Automate backups | Data loss without backups is unrecoverable |
| Restore from backups | Disaster recovery requires tested restore procedures |
| Monitor DB performance | Slow queries and high connections affect production |
| Manage DB in CI/CD pipelines | Schema migrations must be automated and reversible |

---

## 2. Local vs. Remote Database — Development Strategies

When developers need a database to build and test their code, there are two main approaches — each with real trade-offs.

### Strategy 1 — Local Database (Each Developer Installs Their Own)

```
Developer A              Developer B              Developer C
┌─────────────┐          ┌─────────────┐          ┌─────────────┐
│  App Code   │          │  App Code   │          │  App Code   │
│  MySQL 8.0  │          │  MySQL 8.0  │          │  MySQL 8.0  │
│  test data  │          │  test data  │          │  test data  │
└─────────────┘          └─────────────┘          └─────────────┘
  Isolated                Isolated                  Isolated
  (no conflicts)          (no conflicts)            (no conflicts)
```

**Advantages:**
- Complete isolation — one dev's broken data doesn't affect others
- If the DB gets corrupted, just wipe it and start fresh
- Works fully offline

**Disadvantages:**
- Every developer must install and configure the DB manually
- No real/production-like data — test data must be seeded manually or via scripts
- DB version inconsistencies across developer machines can cause bugs
- Time-consuming setup for new team members

---

### Strategy 2 — Shared Remote Database

```
Developer A  ──┐
Developer B  ──┼──► Remote DB Server (shared)
Developer C  ──┘       │
                    All team members
                    use same DB
                    same data
```

**Advantages:**
- No local installation needed — start coding immediately
- Realistic test data available from day one
- Consistent DB version for all developers

**Disadvantages:**
- One developer's bad query or data change affects everyone
- Requires internet/VPN connection to work
- Must coordinate who is testing what to avoid data conflicts
- Risk of accidentally modifying or deleting shared test data

---

### The Modern Solution — Docker for Local DB

Today, the standard DevOps approach combines the best of both worlds: run the DB locally using **Docker**, so it's isolated AND requires no manual installation:

```bash
# Spin up a MySQL database locally — no installation needed
docker run -d \
  --name local-mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=myapp \
  -p 3306:3306 \
  mysql:8.0

# PostgreSQL
docker run -d \
  --name local-postgres \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 \
  postgres:15

# Start/stop without losing data
docker stop local-mysql
docker start local-mysql

# Wipe completely and start fresh
docker rm -f local-mysql
```

Or with **docker-compose** (store in the project repo so everyone uses the same config):

```yaml
# docker-compose.yml
services:
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: myapp
    ports:
      - "3306:3306"
    volumes:
      - db-data:/var/lib/mysql   # persist data between restarts

volumes:
  db-data:
```

```bash
docker compose up -d    # start DB
docker compose down     # stop DB
docker compose down -v  # stop and wipe all data
```

---

## 3. Configuring Database Connections in Applications

Every programming language has libraries or drivers for connecting to databases. The application code needs to know:

- **Which database** to connect to (host + port)
- **Which database name** to use
- **How to authenticate** (username + password)

### Connection String Format

```
DB_TYPE://USERNAME:PASSWORD@HOST:PORT/DATABASE_NAME

Examples:
mysql://admin:secret@localhost:3306/myapp
postgresql://admin:secret@db.prod.example.com:5432/myapp
mongodb://admin:secret@mongo.prod.example.com:27017/myapp
```

### Language-Specific Connection Examples

**Node.js:**
```javascript
const mysql = require('mysql2');

const connection = mysql.createConnection({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME
});
```

**Python:**
```python
import psycopg2
import os

conn = psycopg2.connect(
    host=os.environ['DB_HOST'],
    user=os.environ['DB_USER'],
    password=os.environ['DB_PASSWORD'],
    dbname=os.environ['DB_NAME']
)
```

### The Golden Rule — Never Hardcode Credentials

```bash
# WRONG — credentials in plain text in code
DB_URL = "mysql://admin:mysecretpassword@db.prod.com/myapp"

# RIGHT — credentials from environment variables
DB_HOST = os.environ['DB_HOST']
DB_PASSWORD = os.environ['DB_PASSWORD']
```

### Use `.env` Files for Local Development

```bash
# .env file (NEVER commit to Git)
DB_HOST=localhost
DB_PORT=3306
DB_NAME=myapp
DB_USER=admin
DB_PASSWORD=secretpassword
```

```bash
# .gitignore — always add .env
echo ".env" >> .gitignore
```

> **Security rule:** Credentials hardcoded in source code end up in Git history, where they are permanently visible — even if you delete them later. Always use environment variables, and for production use a secrets manager (AWS Secrets Manager, HashiCorp Vault, or Kubernetes Secrets).

---

## 4. Database Across Environments

In professional software development, there are typically three separate environments — each with its own database:

```
┌─────────────────────────────────────────────────────────────────┐
│                    ENVIRONMENT HIERARCHY                        │
│                                                                 │
│  DEV                  TEST/STAGING           PRODUCTION         │
│  ─────                ────────────           ──────────         │
│  Developer            QA engineers           Real users         │
│  local machine        staging server         production server  │
│                                                                 │
│  DB: localhost        DB: test.db.company    DB: prod.db.company│
│  Data: dummy/seed     Data: near-real        Data: REAL         │
│  Access: dev team     Access: QA + devs      Access: restricted │
│  Security: low        Security: medium        Security: HIGH    │
│  Backup: optional     Backup: optional        Backup: MANDATORY │
└─────────────────────────────────────────────────────────────────┘
```

### Environment-Specific Configuration

The same application code runs in all three environments — only the database connection details change. This is managed through environment variables:

```bash
# Development .env
DB_HOST=localhost
DB_NAME=myapp_dev
DB_USER=dev_user
DB_PASSWORD=dev_password

# Staging .env
DB_HOST=staging-db.company.com
DB_NAME=myapp_staging
DB_USER=staging_user
DB_PASSWORD=staging_password_secure

# Production (set as secrets — not in .env files)
DB_HOST=prod-db.company.com
DB_NAME=myapp_production
DB_USER=prod_user
DB_PASSWORD=<fetched from secrets manager>
```

### Production Database Rules

- **Never test against the production DB** — a bad query can corrupt real user data
- **Never give developers direct production DB access** — use read replicas for data analysis
- **Always use strong passwords and SSL connections** in production
- **Enable audit logging** — track all queries and connection attempts

---

## 5. Database Types

Choosing the right database type for an application is a critical architecture decision. DevOps engineers must understand the options to provision, configure, and manage them correctly.

### Type 1 — Relational Databases (SQL)

Relational databases store data in **structured tables** with rows and columns — like a spreadsheet. Tables relate to each other through **foreign keys**. They use **SQL (Structured Query Language)** for querying.

```
┌────────────────┐         ┌────────────────────┐
│    users       │         │     orders         │
├────────────────┤         ├────────────────────┤
│ id  │ name     │         │ id │ user_id │ amt │
│  1  │ Alice    │◄────────│  1 │    1    │ $50 │
│  2  │ Bob      │         │  2 │    1    │ $30 │
└────────────────┘         │  3 │    2    │ $80 │
                           └────────────────────┘
```

**Key properties — ACID:**
- **Atomicity** — a transaction either fully completes or fully rolls back
- **Consistency** — data always remains in a valid state
- **Isolation** — concurrent transactions don't interfere
- **Durability** — committed data survives crashes

**Popular relational databases:**

| Database | Best For | Notes |
|----------|---------|-------|
| **MySQL** | Web applications, LAMP stack | Most widely used, fast reads |
| **PostgreSQL** | Complex queries, JSON, analytics | Most feature-rich open-source SQL DB |
| **Microsoft SQL Server** | Enterprise Windows environments | Strong Microsoft integration |
| **SQLite** | Embedded apps, mobile, local dev | File-based, no server needed |
| **MariaDB** | MySQL drop-in replacement | Fully open-source fork of MySQL |

**Use when:**
- Data is structured and relationships between tables matter
- You need transactions (banking, e-commerce, inventory)
- Data integrity and consistency are critical
- Complex joins and reporting queries are required

---

### Type 2 — NoSQL Databases (Non-Relational)

NoSQL databases store data in flexible formats — no fixed schema, no tables. Designed for **scale, speed, and flexibility**. There are four main subtypes:

#### 2a — Document Databases

Store data as **JSON-like documents**. Each document can have a different structure.

```json
// A single "user" document — no rigid schema
{
  "_id": "user_001",
  "name": "Alice",
  "email": "alice@example.com",
  "address": {
    "city": "Lahore",
    "country": "Pakistan"
  },
  "orders": [
    { "item": "laptop", "price": 1200 },
    { "item": "mouse",  "price": 25 }
  ]
}
```

| Database | Best For |
|----------|---------|
| **MongoDB** | Content management, catalogs, user profiles, agile development |
| **CouchDB** | Offline-first apps, sync-heavy workloads |
| **Firestore** | Mobile/web apps with Google Firebase |

**Use when:** Data is hierarchical, nested, or varies significantly between records.

---

#### 2b — Key-Value Stores

The simplest NoSQL type — stores data as **key → value** pairs. Extremely fast.

```
"session:user_001"  →  { "logged_in": true, "cart": [...] }
"cache:product_42"  →  { "name": "Laptop", "price": 1200 }
"rate_limit:ip_x"   →  "15"
```

| Database | Best For |
|----------|---------|
| **Redis** | Caching, session storage, rate limiting, pub/sub messaging |
| **Memcached** | Simple distributed caching |
| **DynamoDB** | AWS-native, serverless key-value at massive scale |

**Use when:** You need ultra-fast reads/writes and don't need complex queries. Often used **alongside** a relational DB — the relational DB stores persistent data, Redis caches frequently accessed data.

---

#### 2c — Wide-Column Databases

Store data in **rows with dynamic columns** — optimized for querying huge datasets across distributed nodes.

```
Row key: "user_001"
  Columns: name="Alice", city="Lahore", last_login="2026-08-01"

Row key: "user_002"
  Columns: name="Bob"  (city and last_login columns simply don't exist for this row)
```

| Database | Best For |
|----------|---------|
| **Apache Cassandra** | High-volume writes, time-series data, IoT, globally distributed apps |
| **HBase** | Hadoop ecosystem, big data analytics |
| **Google Bigtable** | Google's managed large-scale column store |

**Use when:** Write throughput is the priority and you have massive datasets across multiple data centers.

---

#### 2d — Graph Databases

Store data as **nodes (entities) and edges (relationships)**. Ideal when relationships between data points are as important as the data itself.

```
(Alice) ──FOLLOWS──► (Bob)
(Alice) ──BOUGHT───► (Laptop)
(Bob)   ──REVIEWED─► (Laptop)
(Alice) ──FRIENDS──► (Carol)
```

| Database | Best For |
|----------|---------|
| **Neo4j** | Social networks, fraud detection, recommendation engines |
| **Amazon Neptune** | AWS-managed graph DB |
| **ArangoDB** | Multi-model: graph + document + key-value |

**Use when:** You have complex, many-to-many relationships — social graphs, recommendation systems, fraud detection networks.

---

### Type 3 — NewSQL Databases

NewSQL combines the best of both worlds:
- **ACID transactions and SQL** from relational DBs
- **Horizontal scalability** from NoSQL

```
Traditional SQL:   Strong consistency + SQL  →  BUT limited to one server
NoSQL:             Scales across many servers →  BUT weaker consistency
NewSQL:            Strong consistency + SQL  +  Scales across many servers ✓
```

| Database | Best For |
|----------|---------|
| **CockroachDB** | Distributed SQL, multi-region, global apps |
| **Google Spanner** | Planet-scale transactions, Google infrastructure |
| **TiDB** | MySQL-compatible, horizontally scalable |
| **PlanetScale** | MySQL-compatible, serverless, automatic sharding |

**Use when:** You need relational data with SQL but expect your application to grow beyond what a single database server can handle.

---

### Type 4 — Time-Series Databases

Optimized for storing and querying **data points indexed by time** — sensor readings, metrics, logs, financial data.

```
timestamp          | metric      | value
2026-08-15 10:00   | cpu_usage   | 45%
2026-08-15 10:01   | cpu_usage   | 67%
2026-08-15 10:02   | cpu_usage   | 82%
```

| Database | Best For |
|----------|---------|
| **InfluxDB** | Application metrics, IoT sensor data |
| **Prometheus** | Kubernetes and infrastructure monitoring (scrapes metrics) |
| **TimescaleDB** | PostgreSQL extension for time-series data |
| **Grafana** | Visualization layer on top of time-series databases |

**Use when:** You are storing metrics, monitoring data, IoT sensor readings, or any data where time is the primary axis.

---

### Database Type Summary

| Type | Data Model | Best For | Examples |
|------|-----------|---------|---------|
| **Relational (SQL)** | Tables, rows, columns | Transactions, structured data | MySQL, PostgreSQL |
| **Document** | JSON documents | Flexible schemas, nested data | MongoDB, Firestore |
| **Key-Value** | Key → Value pairs | Caching, sessions, speed | Redis, DynamoDB |
| **Wide-Column** | Rows with dynamic columns | Massive write throughput, IoT | Cassandra, HBase |
| **Graph** | Nodes and edges | Relationships, social networks | Neo4j, Neptune |
| **NewSQL** | Tables + distributed | SQL at scale | CockroachDB, Spanner |
| **Time-Series** | Timestamped data points | Metrics, monitoring, IoT | InfluxDB, Prometheus |

---

## 6. DevOps Responsibilities for Databases

As a DevOps engineer, your database responsibilities go beyond just "knowing what a database is":

### Installation & Configuration

```bash
# Install MySQL on Ubuntu server
sudo apt update
sudo apt install -y mysql-server

# Secure the installation (set root password, remove test data)
sudo mysql_secure_installation

# Start and enable MySQL
sudo systemctl start mysql
sudo systemctl enable mysql

# Check status
sudo systemctl status mysql
```

### Creating Users & Granting Permissions

```sql
-- Create a dedicated application user (never use root for apps)
CREATE USER 'appuser'@'%' IDENTIFIED BY 'strong_password_here';

-- Grant only the permissions needed
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp.* TO 'appuser'@'%';

-- Apply changes
FLUSH PRIVILEGES;

-- Create a read-only user for monitoring/reporting
CREATE USER 'readonly'@'%' IDENTIFIED BY 'readonly_password';
GRANT SELECT ON myapp.* TO 'readonly'@'%';
```

### Monitoring Database Health

```bash
# Check active connections
SHOW STATUS LIKE 'Threads_connected';

# Check slow queries
SHOW VARIABLES LIKE 'slow_query_log';

# Check DB size
SELECT table_schema AS 'Database',
       ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.TABLES
GROUP BY table_schema;
```

---

## 7. Database Replication

**Replication** copies data from one database server (primary) to one or more others (replicas/secondaries) automatically and continuously.

```
┌──────────────────────────────────────────────────────────────┐
│                   DATABASE REPLICATION                       │
│                                                              │
│   PRIMARY (Master)          REPLICAS (Secondaries)           │
│   ────────────────          ────────────────────────         │
│   Handles all WRITES        Handle READ queries              │
│        │                                                     │
│        │── replicates changes ──►  Replica 1 (same region)   │
│        │                                                     │
│        └── replicates changes ──►  Replica 2 (other region)  │
│                                        (disaster recovery)   │
└──────────────────────────────────────────────────────────────┘
```

### Why Replication Matters

| Benefit | Description |
|---------|-------------|
| **High Availability** | If primary fails, a replica promotes to primary — minimal downtime |
| **Read Scalability** | Spread read queries across replicas — reduce primary load |
| **Disaster Recovery** | Replica in a different region survives a regional outage |
| **Zero-Downtime Backups** | Take backups from replica — no impact on primary performance |

### Types of Replication

```
Synchronous:  Primary waits for replica to confirm before responding
              → No data loss, but slower writes
              → Used in: banking, financial systems

Asynchronous: Primary responds immediately, replica catches up later
              → Faster writes, tiny risk of data loss on primary crash
              → Used in: most web applications
```

---

## 8. Database Backups

**A backup without a tested restore is not a backup.** Backups are insurance — you hope you never need them, but they must work when you do.

### Backup Types

| Type | Description | Speed | Storage |
|------|-------------|-------|---------|
| **Full backup** | Complete copy of entire database | Slow | Large |
| **Incremental** | Only changes since last backup | Fast | Small |
| **Differential** | Changes since last full backup | Medium | Medium |
| **Logical backup** | SQL dump — portable, human-readable | Medium | Medium |
| **Physical backup** | Copy of raw database files | Fastest | Large |

### MySQL Backup Commands

```bash
# Full logical backup of all databases
mysqldump -u root -p --all-databases > backup_$(date +%Y%m%d).sql

# Backup a specific database
mysqldump -u root -p myapp > myapp_backup_$(date +%Y%m%d_%H%M%S).sql

# Backup with compression (saves disk space)
mysqldump -u root -p myapp | gzip > myapp_backup_$(date +%Y%m%d).sql.gz

# Backup specific tables only
mysqldump -u root -p myapp users orders > partial_backup.sql
```

### PostgreSQL Backup Commands

```bash
# Backup a single database
pg_dump -U postgres myapp > myapp_backup_$(date +%Y%m%d).sql

# Backup in compressed custom format (faster restore)
pg_dump -U postgres -Fc myapp > myapp_backup_$(date +%Y%m%d).dump

# Backup all databases
pg_dumpall -U postgres > all_databases_backup.sql
```

### Automated Backup Script

```bash
#!/bin/bash
# db-backup.sh — run via cron for automated daily backups

DB_NAME="myapp"
DB_USER="backup_user"
BACKUP_DIR="/backups/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# Create backup directory
mkdir -p $BACKUP_DIR

# Perform backup
mysqldump -u $DB_USER -p$DB_PASSWORD $DB_NAME | \
    gzip > "$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"

echo "Backup completed: ${DB_NAME}_${DATE}.sql.gz"

# Remove backups older than retention period
find $BACKUP_DIR -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete
echo "Old backups cleaned up."
```

```bash
# Schedule via cron — runs daily at 2:00 AM
crontab -e
# Add:
0 2 * * * /scripts/db-backup.sh >> /var/log/db-backup.log 2>&1
```

### Backup Storage Best Practice — 3-2-1 Rule

```
3 copies of data
  2 on different storage media (disk + cloud)
    1 offsite (different region or provider)

Example:
  - Local disk backup (fast restore)
  - AWS S3 bucket (durable cloud storage)
  - AWS S3 in a different region (geographic redundancy)
```

---

## 9. Database Restore & Disaster Recovery

A backup is only valuable if you can successfully restore from it. **Test your restores regularly** — not just during an actual emergency.

### MySQL Restore

```bash
# Restore from a .sql backup
mysql -u root -p myapp < myapp_backup_20260815.sql

# Restore from a compressed backup
gunzip < myapp_backup_20260815.sql.gz | mysql -u root -p myapp

# Restore to a different database (useful for testing restore without affecting prod)
mysql -u root -p new_test_db < myapp_backup_20260815.sql
```

### PostgreSQL Restore

```bash
# Restore from SQL dump
psql -U postgres myapp < myapp_backup_20260815.sql

# Restore from custom format dump
pg_restore -U postgres -d myapp myapp_backup_20260815.dump

# Restore to a new database (for testing)
createdb -U postgres myapp_restore_test
pg_restore -U postgres -d myapp_restore_test myapp_backup_20260815.dump
```

### Key Recovery Concepts

| Term | Definition |
|------|-----------|
| **RPO (Recovery Point Objective)** | Maximum acceptable data loss — how old can the backup be? e.g., "max 1 hour of data loss" |
| **RTO (Recovery Time Objective)** | Maximum acceptable downtime — how long to restore? e.g., "must be back up in 30 minutes" |
| **Point-in-Time Recovery** | Restore to a specific moment — useful for recovering from accidental data deletion |
| **Failover** | Automatically switch from failed primary to a replica |

---

## 10. Database in CI/CD Pipelines

Database changes (schema migrations) must be automated and go through the same pipeline as application code.

### The Problem

```
Developer changes the DB schema (adds a column, renames a table)
→ Deploys new app code that expects the new schema
→ But the database still has the old schema
→ Application crashes
```

### The Solution — Database Migrations

Migration tools version-control your database schema alongside your code:

```bash
# Popular migration tools:
# - Flyway (Java / any DB)
# - Liquibase (Java / any DB)
# - Alembic (Python / SQLAlchemy)
# - Sequelize migrations (Node.js)
# - Rails ActiveRecord migrations (Ruby)
# - Prisma migrate (Node.js)
```

### Example Migration File (Flyway)

```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- V2__add_username_to_users.sql
ALTER TABLE users ADD COLUMN username VARCHAR(100);

-- V3__add_orders_table.sql
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    amount DECIMAL(10,2),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### In the CI/CD Pipeline

```yaml
# In your GitHub Actions or Jenkins pipeline
steps:
  - name: Run database migrations
    run: flyway migrate
    env:
      FLYWAY_URL: ${{ secrets.DB_URL }}
      FLYWAY_USER: ${{ secrets.DB_USER }}
      FLYWAY_PASSWORD: ${{ secrets.DB_PASSWORD }}

  - name: Deploy application
    run: ./deploy.sh
    # Application deploys AFTER migrations — schema is ready
```

---

## 11. Quick Reference Cheat Sheet

### Local Dev Database (Docker)

```bash
# MySQL
docker run -d --name mysql-dev \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=myapp \
  -p 3306:3306 mysql:8.0

# PostgreSQL
docker run -d --name pg-dev \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 postgres:15
```

### MySQL Admin Commands

```sql
SHOW DATABASES;                    -- list databases
USE myapp;                         -- select database
SHOW TABLES;                       -- list tables
SHOW STATUS LIKE 'Threads%';       -- connection stats
SHOW PROCESSLIST;                  -- active queries
```

### Backup & Restore

```bash
# MySQL backup
mysqldump -u root -p myapp > backup.sql
mysqldump -u root -p myapp | gzip > backup.sql.gz

# MySQL restore
mysql -u root -p myapp < backup.sql
gunzip < backup.sql.gz | mysql -u root -p myapp

# PostgreSQL backup
pg_dump -U postgres myapp > backup.sql
pg_dump -U postgres -Fc myapp > backup.dump

# PostgreSQL restore
psql -U postgres myapp < backup.sql
pg_restore -U postgres -d myapp backup.dump
```

### Database Type Quick Picker

| Need | Use |
|------|-----|
| Structured data + transactions | MySQL / PostgreSQL |
| Flexible/nested JSON data | MongoDB |
| Caching / sessions / speed | Redis |
| Massive write throughput | Cassandra |
| Complex relationships | Neo4j |
| SQL + horizontal scale | CockroachDB |
| Metrics / monitoring data | InfluxDB / Prometheus |

### Security Rules

```
✅ Always use environment variables for credentials
✅ Never commit .env files to Git
✅ Create dedicated DB users per application (not root)
✅ Grant minimum required permissions only
✅ Use SSL/TLS for DB connections in production
✅ Store production credentials in a secrets manager
✅ Restrict production DB access to application servers only
```

---

*End of Lesson 22*