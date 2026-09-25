# SQL, The Complete Guide

A thorough, zero-to-professional tour of SQL: what relational databases are, how every major clause works, how to combine tables, how to analyze data with window functions, how to design schemas that stay correct, and how SQL is actually used in production by application developers, analysts, data scientists, data engineers, and AI engineers. Every concept is paired with a query against one small sample database and the real output that query produces, so nothing is left abstract.

The examples target **PostgreSQL**, the most standards-faithful open-source database and the default choice for new projects. Every query in this guide was executed on PostgreSQL 16, and the outputs shown are the real results. Everything works on 16, 17, and 18 (18 is the current stable series, with 19 in beta as of September 2026), though `EXPLAIN` plans and timings will look slightly different on other versions and machines. Where another database spells something differently, that is flagged inline, and [Section 54](#54-dialect-translation-table) collects the differences in one table.

**Sources:**
- [PostgreSQL Documentation](https://www.postgresql.org/docs/current/) — the authoritative reference for every statement and function used here
- [PostgreSQL Tutorial (official)](https://www.postgresql.org/docs/current/tutorial.html) — the project's own introduction
- [SQLite](https://sqlite.org/lang.html), [MySQL](https://dev.mysql.com/doc/refman/8.4/en/), and [SQL Server](https://learn.microsoft.com/en-us/sql/t-sql/language-reference) language references — for the dialect notes
- [DuckDB Documentation](https://duckdb.org/docs/) and [pgvector](https://github.com/pgvector/pgvector) — analytical and vector-search extensions of the ecosystem
- [psycopg 3](https://www.psycopg.org/psycopg3/docs/), [SQLAlchemy 2.0](https://docs.sqlalchemy.org/en/20/), and [pandas I/O](https://pandas.pydata.org/docs/user_guide/io.html#sql-queries) — SQL from Python
- [W3Schools SQL](https://www.w3schools.com/sql/) — worked examples and a keyword reference

---

## How to read this guide

The sections build strictly on each other. First comes the landscape — what SQL is, which database to use, how to install one, and the sample data — then reading data from one table, then the functions that transform values, then aggregation, then combining tables, then window functions and the analysis patterns built on them. After that the guide switches from *reading* data to *owning* it — changing data, designing schemas, indexes, transactions, performance, and security. The second-to-last group shows how SQL fits into real work from Python, pipelines, warehouses, and machine learning systems. The final five sections are pure reference.

If you already know basic `SELECT` queries, skim Sections 1 to 10 and start properly at [Section 16](#16-aggregate-functions). If you are preparing for interviews, Sections 5, 8, 21, 24, 27 to 30, 39, 41, and 52 are where most questions come from.

Three mental models recur and are worth holding from the start:

1. **SQL is declarative.** You describe *what* result you want, never *how* to compute it. The database's query planner decides the how. This is why the same query can go from ten seconds to ten milliseconds when an index is added, without changing a character of it.
2. **Every query takes tables and returns a table.** A `SELECT` produces a result set that has rows and columns just like a stored table, which is why queries can be nested inside other queries, joined to tables, and saved as views.
3. **The clauses run in a different order than you write them.** `FROM` runs first and `SELECT` runs nearly last. Almost every "why can't I use my alias here?" error comes from forgetting this ([Section 5](#5-how-a-query-runs-written-order-vs-logical-order)).

Each section ends with a short **Notes** list of the things that trip people up or that interviewers like to ask about. The reference sections at the end are the exception, since every line in them is already a note.

To follow along, install PostgreSQL ([Section 3](#3-setup-postgresql-psql-and-clients)), load the script in [Section 4](#4-the-sample-database), and run each query yourself. Typing queries and predicting their output before you run them is the fastest way to learn SQL.

---

## Table of Contents

**Foundations**
- [1. What Databases and SQL Are](#1-what-databases-and-sql-are)
- [2. The SQL Landscape: Dialects, Engines, and When to Use Which](#2-the-sql-landscape-dialects-engines-and-when-to-use-which)
- [3. Setup: PostgreSQL, psql, and Clients](#3-setup-postgresql-psql-and-clients)
- [4. The Sample Database](#4-the-sample-database)
- [5. How a Query Runs: Written Order vs Logical Order](#5-how-a-query-runs-written-order-vs-logical-order)

**Querying One Table**
- [6. SELECT, FROM, and Aliases](#6-select-from-and-aliases)
- [7. Filtering with WHERE](#7-filtering-with-where)
- [8. NULL and Three-Valued Logic](#8-null-and-three-valued-logic)
- [9. ORDER BY, LIMIT, and DISTINCT](#9-order-by-limit-and-distinct)
- [10. Expressions and CASE](#10-expressions-and-case)

**Types and Functions**
- [11. Data Types and Casting](#11-data-types-and-casting)
- [12. Strings and Pattern Matching](#12-strings-and-pattern-matching)
- [13. Numbers and Math](#13-numbers-and-math)
- [14. Dates, Times, and Intervals](#14-dates-times-and-intervals)
- [15. COALESCE, NULLIF, GREATEST, LEAST](#15-coalesce-nullif-greatest-least)

**Aggregation**
- [16. Aggregate Functions](#16-aggregate-functions)
- [17. GROUP BY and HAVING](#17-group-by-and-having)
- [18. FILTER, ROLLUP, CUBE, and GROUPING SETS](#18-filter-rollup-cube-and-grouping-sets)

**Combining Tables**
- [19. Keys and Relationships](#19-keys-and-relationships)
- [20. INNER JOIN](#20-inner-join)
- [21. Outer Joins](#21-outer-joins)
- [22. Self, Cross, Semi, Anti, and Lateral Joins](#22-self-cross-semi-anti-and-lateral-joins)
- [23. UNION, INTERSECT, EXCEPT](#23-union-intersect-except)
- [24. Subqueries](#24-subqueries)
- [25. Common Table Expressions](#25-common-table-expressions)
- [26. Recursive CTEs](#26-recursive-ctes)

**Window Functions and Analytics**
- [27. Window Function Basics](#27-window-function-basics)
- [28. Ranking and Offset Functions](#28-ranking-and-offset-functions)
- [29. Frames, Running Totals, and Moving Averages](#29-frames-running-totals-and-moving-averages)
- [30. Classic Patterns: Top-N, Dedup, Gaps and Islands, Pivots](#30-classic-patterns-top-n-dedup-gaps-and-islands-pivots)
- [31. Analytics Playbook: Cohorts, Retention, Funnels, Growth](#31-analytics-playbook-cohorts-retention-funnels-growth)

**Changing Data**
- [32. INSERT](#32-insert)
- [33. UPDATE, DELETE, and TRUNCATE](#33-update-delete-and-truncate)
- [34. Upserts and MERGE](#34-upserts-and-merge)

**Designing Databases**
- [35. Creating and Altering Tables](#35-creating-and-altering-tables)
- [36. Constraints and Referential Integrity](#36-constraints-and-referential-integrity)
- [37. Normalization and Schema Design](#37-normalization-and-schema-design)
- [38. Views and Materialized Views](#38-views-and-materialized-views)
- [39. Indexes](#39-indexes)
- [40. JSON, Arrays, and Full-Text Search](#40-json-arrays-and-full-text-search)

**Production SQL**
- [41. Transactions, Isolation, and Locking](#41-transactions-isolation-and-locking)
- [42. Query Plans and Performance](#42-query-plans-and-performance)
- [43. Functions, Procedures, and Triggers](#43-functions-procedures-and-triggers)
- [44. Roles, Permissions, and Row-Level Security](#44-roles-permissions-and-row-level-security)
- [45. Import, Export, and Backup](#45-import-export-and-backup)

**SQL in the Data and AI Stack**
- [46. SQL from Python: Drivers, Parameters, and Injection](#46-sql-from-python-drivers-parameters-and-injection)
- [47. ORMs, SQLAlchemy, and pandas](#47-orms-sqlalchemy-and-pandas)
- [48. Data Engineering: Warehouses, Star Schemas, and Pipelines](#48-data-engineering-warehouses-star-schemas-and-pipelines)
- [49. Analytical Engines: DuckDB and Cloud Warehouses](#49-analytical-engines-duckdb-and-cloud-warehouses)
- [50. SQL for Machine Learning and AI: Features, Splits, and Vector Search](#50-sql-for-machine-learning-and-ai-features-splits-and-vector-search)

**Reference**
- [51. Common Mistakes](#51-common-mistakes)
- [52. Interview Quick-Fire](#52-interview-quick-fire)
- [53. Cheat Sheet](#53-cheat-sheet)
- [54. Dialect Translation Table](#54-dialect-translation-table)
- [55. Keyword Reference](#55-keyword-reference)

---

## 1. What Databases and SQL Are

A **database** is an organized, persistent collection of data managed by a program called a **database management system** (DBMS). The DBMS handles the hard parts that you would otherwise write yourself — storing data safely on disk, letting many users read and write at once without corrupting anything, enforcing rules about what data is valid, and finding the rows you ask for quickly.

A **relational database** (managed by an RDBMS) stores data in **tables**. A table has named, typed **columns** and any number of **rows**. Each row is one record: one customer, one order, one product. The word *relational* comes from the mathematical term *relation* (a set of tuples), not from "relationships between tables", although tables do relate to each other through shared key values.

```text
customers                                  orders
┌─────────────┬────────────┬─────────┐     ┌──────────┬─────────────┬───────────┐
│ customer_id │ first_name │ country │     │ order_id │ customer_id │ status    │
├─────────────┼────────────┼─────────┤     ├──────────┼─────────────┼───────────┤
│           1 │ Alice      │ UK      │◀────│        1 │           1 │ delivered │
│           2 │ Bruno      │ Portugal│◀─┬──│        2 │           2 │ delivered │
│           3 │ Chloe      │ Canada  │  └──│       11 │           2 │ delivered │
└─────────────┴────────────┴─────────┘     └──────────┴─────────────┴───────────┘
  primary key: customer_id                   foreign key: orders.customer_id → customers
```

**SQL** (Structured Query Language, pronounced "S-Q-L" or "sequel", both are fine) is the language you use to talk to a relational database. It was developed in the early 1970s (originally named SEQUEL), became an ANSI standard in 1986 and an ISO standard in 1987, and has been revised many times since (SQL:1999 added recursive queries, SQL:2003 added window functions, SQL:2016 added JSON, SQL:2023 added property graph queries). Every major relational database speaks SQL, with its own extensions and quirks on top of the standard.

SQL statements fall into five families. You will see these abbreviations in documentation and job descriptions:

| Family | Stands for | Statements | Purpose |
|:--|:--|:--|:--|
| DQL | Data Query Language | `SELECT` | Read data |
| DML | Data Manipulation Language | `INSERT`, `UPDATE`, `DELETE`, `MERGE` | Change rows |
| DDL | Data Definition Language | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | Define structure |
| DCL | Data Control Language | `GRANT`, `REVOKE` | Control access |
| TCL | Transaction Control Language | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` | Group changes atomically |

Some texts fold DQL into DML, since `SELECT` technically manipulates nothing. The distinction matters in practice mostly for permissions — an analyst typically gets DQL only, an application gets DQL and DML, and only migrations or administrators run DDL and DCL.

Why use a database instead of CSV files or Python dictionaries? Because a database gives you four guarantees that files do not:

- **Integrity.** Rules such as "every order must belong to an existing customer" and "price must be positive" are enforced on every write, by the database, forever.
- **Concurrency.** Hundreds of connections can read and write simultaneously and each sees a consistent picture.
- **Durability.** Once a change is committed, it survives crashes and power loss.
- **Declarative querying.** You ask a question in SQL and the database works out an efficient way to answer it, using indexes and statistics you never see.

Relational databases are not the only kind. **NoSQL** databases trade some of these guarantees for other strengths — document stores (MongoDB) for flexible nested records, key-value stores (Redis) for extreme speed on simple lookups, wide-column stores (Cassandra) for massive write throughput, graph databases (Neo4j) for traversing relationships, and vector databases (Pinecone, Qdrant) for similarity search on embeddings. In practice most systems use a relational database as the source of truth and add a specialized store only where a clear need appears — and PostgreSQL itself covers many of those needs through JSON columns ([Section 40](#40-json-arrays-and-full-text-search)) and the pgvector extension ([Section 50](#50-sql-for-machine-learning-and-ai-features-splits-and-vector-search)).

**Notes:**
- A *database* is the data; a *DBMS* is the software (PostgreSQL, MySQL). People say "the database" for both, and that is fine in conversation.
- "Relational" refers to tables as mathematical relations. Rows in a table have no inherent order, which is why you must use `ORDER BY` whenever order matters.
- SQL is a standard, but no database implements all of it and every database adds extensions. Roughly 90 percent of what you learn transfers directly ([Section 2](#2-the-sql-landscape-dialects-engines-and-when-to-use-which)).
- SQL keywords are case-insensitive: `select`, `SELECT`, and `SeLeCt` are identical. The convention in this guide (uppercase keywords, lowercase names) is for readability only.

---

## 2. The SQL Landscape: Dialects, Engines, and When to Use Which

"SQL" is one language with many **dialects**. Each database product implements the ISO standard core and then adds its own syntax, functions, and procedural language. Learning SQL on one database and moving to another feels like moving between British and American English — the grammar is the same, most vocabulary is the same, and a handful of words and spellings differ.

The engines you will meet split into two broad families, depending on the workload they are built for:

- **OLTP** (Online Transaction Processing) databases run applications. They handle many small, concurrent reads and writes ("insert this order", "fetch this user"), store data row by row, and enforce constraints strictly. PostgreSQL, MySQL, SQL Server, Oracle, and SQLite are OLTP databases.
- **OLAP** (Online Analytical Processing) engines run analysis. They handle few, huge, read-mostly queries ("revenue by country by month over five years"), store data column by column so they can scan billions of values fast, and usually live in a separate data warehouse fed from the OLTP systems. BigQuery, Snowflake, Redshift, Databricks SQL, ClickHouse, and DuckDB are OLAP engines.

### The major engines

| Engine | Dialect | Type | Typical use | Pick it when |
|:--|:--|:--|:--|:--|
| PostgreSQL | PostgreSQL (with PL/pgSQL) | OLTP, open source | Web and API backends, general purpose, geospatial (PostGIS), vector search (pgvector) | Starting almost any new project; you want standards compliance and rich features |
| MySQL / MariaDB | MySQL | OLTP, open source | Web applications, especially the PHP and WordPress world | Joining a stack that already uses it, or needing its specific hosting ecosystem |
| SQLite | SQLite | Embedded, single file | Mobile apps, desktop apps, tests, prototypes, local tools | You want a database with zero server, stored in one file next to your code |
| SQL Server | T-SQL (Transact-SQL) | OLTP, commercial (Microsoft) | Enterprise and .NET applications, Microsoft BI stacks | Working in a Microsoft or Azure enterprise environment |
| Oracle | Oracle SQL with PL/SQL | OLTP, commercial | Large enterprises, banking, telecoms, ERP systems | The organization already runs on Oracle |
| DuckDB | PostgreSQL-flavored | OLAP, embedded | Local analytics on CSV and Parquet files, notebooks, data science | You want warehouse-speed analysis on your laptop with no server ([Section 49](#49-analytical-engines-duckdb-and-cloud-warehouses)) |
| BigQuery | GoogleSQL | OLAP, cloud warehouse | Analytics on Google Cloud, serverless, pay per data scanned | The data lives in Google Cloud |
| Snowflake | Snowflake SQL | OLAP, cloud warehouse | Company-wide analytics, data sharing, runs on any major cloud | A company-wide warehouse separate from any one cloud |
| Redshift | PostgreSQL-derived | OLAP, cloud warehouse | Analytics on AWS | The data lives in AWS |
| Databricks / Spark SQL | Spark SQL | OLAP, lakehouse | Large-scale data engineering and ML on data lakes | Data is huge, file-based, and processed with Spark |
| ClickHouse | ClickHouse SQL | OLAP, open source | Real-time analytics on event and log data | Sub-second aggregation over billions of events |

Hosted versions of the OLTP databases (Amazon RDS and Aurora, Google Cloud SQL, Azure Database, Supabase, Neon) run the same engines, so everything in this guide applies to them unchanged.

### How transferable the skills are

The core of SQL is identical everywhere: `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, joins, subqueries, CTEs, `CASE`, aggregate functions, window functions, `INSERT`, `UPDATE`, `DELETE`, and the relational thinking behind them. That is well over three quarters of this guide, and it is what interviews test. What changes between dialects is a predictable set of surface details:

| Area | What differs | Example |
|:--|:--|:--|
| Limiting rows | `LIMIT` vs `TOP` vs `FETCH FIRST` | `LIMIT 5` (PostgreSQL, MySQL, SQLite) vs `SELECT TOP 5` (SQL Server) |
| Auto-increment keys | Identity syntax | `GENERATED AS IDENTITY` vs `AUTO_INCREMENT` vs `IDENTITY(1,1)` |
| String concatenation | Operator vs function | `'a' \|\| 'b'` vs `CONCAT('a', 'b')` vs `'a' + 'b'` |
| Date and time functions | Names and arguments | `date_trunc('month', d)` vs `DATE_FORMAT(d, '%Y-%m-01')` vs `DATETRUNC(month, d)` |
| Upserts | Conflict syntax | `ON CONFLICT` vs `ON DUPLICATE KEY UPDATE` vs `MERGE` |
| Identifier quoting | Quote characters | `"name"` vs `` `name` `` vs `[name]` |
| Procedural code | Language for functions and procedures | PL/pgSQL vs T-SQL vs PL/SQL |
| Extras | Features only some engines have | `ILIKE`, `DISTINCT ON`, `QUALIFY`, `RETURNING` |

A practical rule: if you know PostgreSQL well, you can be productive in any other dialect within a day, looking up the differences as you hit them. PostgreSQL is the best one to learn first because it follows the standard most closely, so fewer of your habits need unlearning later, and because DuckDB and Redshift deliberately copy much of its syntax.

### SQL-adjacent languages

Some tools use SQL-like syntax on non-relational data: **HiveQL** and **Spark SQL** query files in data lakes, **Kusto (KQL)** queries logs in Azure, **PromQL** queries metrics, and **Cypher** queries graph databases. Pandas and Polars express the same relational operations (filter, join, group, aggregate) as method calls. The relational thinking this guide teaches transfers to all of them even where the syntax does not.

**Notes:**
- "Which SQL should I learn?" has a clear answer — learn standard SQL on PostgreSQL. Dialect differences are a lookup problem, not a learning problem.
- OLTP vs OLAP is a frequent interview question. The one-line answer: OLTP is many small transactions on current data, row-oriented; OLAP is few large analytical scans over history, column-oriented.
- SQLite is dynamically typed by default (it will store the text `'abc'` in an integer column unless the table is declared `STRICT`). Do not let SQLite habits convince you that types are optional.
- Oracle treats the empty string `''` as `NULL`. No other major database does. This single quirk breaks a surprising amount of ported code.
- MySQL lacks `FULL OUTER JOIN` entirely — emulate it with a `UNION` of a left and a right join ([Section 21](#21-outer-joins)).

---

## 3. Setup: PostgreSQL, psql, and Clients

You need a running PostgreSQL server and a client to send it SQL. Pick one installation route:

| Platform | Command or installer |
|:--|:--|
| macOS | `brew install postgresql@18` then `brew services start postgresql@18`, or the Postgres.app bundle |
| Ubuntu / Debian | `sudo apt install postgresql` (the server starts automatically) |
| Windows | The EDB installer from postgresql.org, which includes pgAdmin |
| Any OS with Docker | `docker run --name pg -e POSTGRES_PASSWORD=secret -p 5432:5432 -d postgres:18` |
| No install at all | A free hosted database (Supabase, Neon) or an online playground |

Docker is the cleanest option if you already have it — the whole server lives in one disposable container, and `docker rm -f pg` removes every trace.

### psql, the command-line client

`psql` ships with PostgreSQL and is worth learning even if you prefer a GUI, because it is available on every server you will ever SSH into. Connect with:

```bash
psql -h localhost -U postgres              # connect as user postgres (prompts for the password)
psql -h localhost -U postgres -d shop      # connect straight to the shop database
psql "postgresql://postgres:secret@localhost:5432/shop"   # the same, as a connection URL
```

Inside `psql`, lines ending with `;` are SQL sent to the server. Lines starting with a backslash are **meta-commands** handled by `psql` itself, and they need no semicolon:

| Meta-command | What it does |
|:--|:--|
| `\l` | List databases |
| `\c shop` | Connect to the `shop` database |
| `\dt` | List tables in the current database |
| `\d products` | Describe a table: columns, types, indexes, constraints |
| `\dn`, `\dv`, `\df`, `\du` | List schemas, views, functions, roles |
| `\x` | Toggle expanded display (one column per line, good for wide rows) |
| `\timing` | Show how long each query takes |
| `\i file.sql` | Run a SQL file |
| `\e` | Open the last query in your editor |
| `\?` and `\h SELECT` | Help for meta-commands, and for any SQL statement |
| `\q` | Quit |

### GUI clients

For exploring data, a graphical client is more comfortable. **DBeaver** (free, works with every database), **pgAdmin** (free, PostgreSQL-specific), **DataGrip** (paid, JetBrains), and database extensions for VS Code all do the same job: a connection manager, a schema browser, a query editor with autocompletion, and a results grid.

### Style conventions

SQL ignores whitespace and keyword case, so style is purely for humans. The conventions used here are the most common in industry:

```sql
-- Single-line comments start with two dashes.
/* Block comments
   span lines. */

SELECT first_name,              -- keywords in UPPERCASE
       last_name                -- one column per line when there are several
FROM customers                  -- names in lowercase snake_case
WHERE country = 'UK'            -- string literals in single quotes
ORDER BY last_name;             -- every statement ends with a semicolon
```

**Notes:**
- Single quotes are for **string values** (`'UK'`). Double quotes are for **identifiers** such as table and column names (`"Order Date"`). Mixing them up is the most common beginner syntax error in PostgreSQL.
- Unquoted identifiers are folded to lowercase, so `SELECT * FROM Customers` reads the table `customers`. If a table was created as `"Customers"` with quotes, you must quote it forever. Avoid quoted, mixed-case names entirely.
- The semicolon terminates a statement. In `psql`, a query without one simply waits for more input, which is why the prompt changes from `shop=#` to `shop-#`.
- Never develop against a production database. A local or Docker instance costs nothing and cannot hurt anyone.

---

## 4. The Sample Database

Every example in this guide queries one small database — an online shop. It is deliberately small enough that you can check every result by eye, yet it contains all the awkward cases that real data has: missing values, customers who never ordered, products that never sold, prices that changed over time, a category tree, a management hierarchy, and salary ties.

```text
            ┌──────────────┐
            │  categories  │──┐ parent_id (a tree: Electronics > Computers > Laptops)
            └──────┬───────┘◀─┘
                   │ 1
                   │ *
┌───────────┐    ┌─┴──────────┐     ┌───────────────┐
│ customers │    │  products  │     │   employees   │──┐ manager_id (a hierarchy)
└─────┬─────┘    └─────┬──────┘     └───────┬───────┘◀─┘
      │ 1              │ 1                  │ 1 (sales rep, optional)
      │ *              │ *                  │ *
┌─────┴─────┐ 1  * ┌───┴─────────┐          │
│  orders   ├──────┤ order_items │          │
└─────┬─────┘      └─────────────┘          │
      └─────────────────────────────────────┘
```

| Table | Rows | One row is | Interesting features |
|:--|:--|:--|:--|
| `customers` | 10 | A registered customer | Two have no city (`NULL`); two have never ordered |
| `categories` | 7 | A product category | `parent_id` forms a tree; `Home` has no products |
| `products` | 12 | A product for sale | `jsonb` attributes; one out of stock; two never sold |
| `employees` | 9 | A staff member | `manager_id` forms a hierarchy; salary ties at 95,000 and 60,000 |
| `orders` | 15 | One checkout | January to June 2026; four statuses; two with no sales rep |
| `order_items` | 26 | One product line within an order | Composite primary key; `unit_price` records the price at the time of sale |

### The full script

Save this as `shop.sql`, create the database, and load it:

```bash
psql -h localhost -U postgres -c "CREATE DATABASE shop"
psql -h localhost -U postgres -d shop -f shop.sql
```

```sql
-- ============================================================
--  shop: the sample database used throughout this guide
-- ============================================================

CREATE TABLE customers (
    customer_id  integer GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    first_name   text        NOT NULL,
    last_name    text        NOT NULL,
    email        text        NOT NULL UNIQUE,
    city         text,                      -- nullable on purpose
    country      text        NOT NULL,
    signup_date  date        NOT NULL
);

CREATE TABLE categories (
    category_id  integer GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    name         text        NOT NULL UNIQUE,
    parent_id    integer     REFERENCES categories (category_id)   -- a tree
);

CREATE TABLE products (
    product_id   integer GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    name         text          NOT NULL,
    category_id  integer       NOT NULL REFERENCES categories (category_id),
    price        numeric(10,2) NOT NULL CHECK (price > 0),
    stock        integer       NOT NULL DEFAULT 0 CHECK (stock >= 0),
    attributes   jsonb                      -- flexible, per-product details
);

CREATE TABLE employees (
    employee_id  integer GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    first_name   text          NOT NULL,
    last_name    text          NOT NULL,
    title        text          NOT NULL,
    department   text          NOT NULL,
    manager_id   integer       REFERENCES employees (employee_id),  -- self-reference
    salary       numeric(10,2) NOT NULL,
    hire_date    date          NOT NULL
);

CREATE TABLE orders (
    order_id     integer GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    customer_id  integer     NOT NULL REFERENCES customers (customer_id),
    employee_id  integer     REFERENCES employees (employee_id),    -- sales rep, optional
    ordered_at   timestamptz NOT NULL,
    status       text        NOT NULL
                 CHECK (status IN ('pending', 'shipped', 'delivered', 'cancelled'))
);

CREATE TABLE order_items (
    order_id     integer       NOT NULL REFERENCES orders (order_id),
    product_id   integer       NOT NULL REFERENCES products (product_id),
    quantity     integer       NOT NULL CHECK (quantity > 0),
    unit_price   numeric(10,2) NOT NULL,     -- price at time of sale
    PRIMARY KEY (order_id, product_id)
);

INSERT INTO customers (first_name, last_name, email, city, country, signup_date) VALUES
    ('Alice',  'Martin', 'alice@example.com',  'London',    'UK',      '2025-01-05'),
    ('Bruno',  'Silva',  'bruno@example.com',  'Lisbon',    'Portugal','2025-01-18'),
    ('Chloe',  'Kim',    'chloe@example.com',  'Toronto',   'Canada',  '2025-02-02'),
    ('Daniel', 'Okafor', 'daniel@example.com', 'Berlin',    'Germany', '2025-02-20'),
    ('Emma',   'Novak',  'emma@example.com',   'Toronto',   'Canada',  '2025-03-11'),
    ('Farid',  'Haddad', 'farid@example.com',  NULL,        'UK',      '2025-03-30'),
    ('Grace',  'Liu',    'grace@example.com',  'Boston',    'USA',     '2025-04-14'),
    ('Hugo',   'Berg',   'hugo@example.com',   'Stockholm', 'Sweden',  '2025-05-01'),
    ('Isla',   'Murphy', 'isla@example.com',   NULL,        'USA',     '2025-05-22'),
    ('Jonas',  'Weber',  'jonas@example.com',  'Berlin',    'Germany', '2025-06-09');

INSERT INTO categories (name, parent_id) VALUES
    ('Electronics', NULL),   -- 1
    ('Computers',   1),      -- 2
    ('Laptops',     2),      -- 3
    ('Accessories', 1),      -- 4
    ('Books',       NULL),   -- 5
    ('Programming', 5),      -- 6
    ('Home',        NULL);   -- 7 (no products yet)

INSERT INTO products (name, category_id, price, stock, attributes) VALUES
    ('Laptop Pro 14',          3, 1299.00,  15, '{"brand": "Nova",   "ram_gb": 16, "color": "silver"}'),
    ('Laptop Air 13',          3,  999.00,  20, '{"brand": "Nova",   "ram_gb": 8,  "color": "gold"}'),
    ('Mechanical Keyboard',    4,   89.90,  50, '{"brand": "Clack",  "switches": "brown"}'),
    ('Wireless Mouse',         4,   29.99, 120, '{"brand": "Clack",  "dpi": 1600}'),
    ('USB-C Hub',              4,   45.00,   0, '{"brand": "Portly", "ports": 7}'),
    ('27in Monitor',           2,  249.00,  30, '{"brand": "Vista",  "size_in": 27}'),
    ('Desktop Tower',          2, 1499.00,   5, '{"brand": "Nova",   "ram_gb": 32}'),
    ('SQL for Everyone',       6,   39.00, 200, '{"pages": 320, "format": "paperback"}'),
    ('Python Deep Dive',       6,   49.00, 150, '{"pages": 540, "format": "hardcover"}'),
    ('Designing Data Systems', 6,   55.00,  80, '{"pages": 610, "format": "paperback"}'),
    ('Noise-Cancelling Headphones', 4, 199.00, 25, '{"brand": "Hush"}'),
    ('Webcam HD',              4,   59.00,  40, NULL);

INSERT INTO employees (first_name, last_name, title, department, manager_id, salary, hire_date) VALUES
    ('Sofia',  'Reyes',  'CEO',             'Executive',   NULL, 180000, '2019-03-01'),  -- 1
    ('Marcus', 'Chen',   'CTO',             'Engineering', 1,    150000, '2019-06-15'),  -- 2
    ('Priya',  'Nair',   'Head of Sales',   'Sales',       1,    120000, '2020-01-10'),  -- 3
    ('Leo',    'Novak',  'Senior Engineer', 'Engineering', 2,    110000, '2020-09-01'),  -- 4
    ('Mia',    'Laine',  'Engineer',        'Engineering', 2,     95000, '2022-02-14'),  -- 5
    ('Omar',   'Farah',  'Engineer',        'Engineering', 4,     95000, '2023-05-02'),  -- 6
    ('Nina',   'Park',   'Sales Rep',       'Sales',       3,     60000, '2021-07-19'),  -- 7
    ('Tom',    'Walsh',  'Sales Rep',       'Sales',       3,     60000, '2024-01-08'),  -- 8
    ('Eva',    'Brandt', 'Support Lead',    'Support',     1,     70000, '2021-11-03');  -- 9

INSERT INTO orders (customer_id, employee_id, ordered_at, status) VALUES
    (1, 7,    '2026-01-05 09:15:00+00', 'delivered'),   -- 1
    (2, 8,    '2026-01-12 14:30:00+00', 'delivered'),   -- 2
    (1, 7,    '2026-01-28 19:45:00+00', 'delivered'),   -- 3
    (3, 7,    '2026-02-03 11:00:00+00', 'delivered'),   -- 4
    (4, 8,    '2026-02-14 16:20:00+00', 'delivered'),   -- 5
    (5, NULL, '2026-02-27 08:05:00+00', 'cancelled'),   -- 6
    (3, 7,    '2026-03-09 13:10:00+00', 'delivered'),   -- 7
    (6, 8,    '2026-03-15 10:40:00+00', 'delivered'),   -- 8
    (1, 8,    '2026-03-30 21:05:00+00', 'delivered'),   -- 9
    (7, 7,    '2026-04-11 12:00:00+00', 'delivered'),   -- 10
    (2, 7,    '2026-04-25 17:25:00+00', 'delivered'),   -- 11
    (5, 8,    '2026-05-06 09:50:00+00', 'delivered'),   -- 12
    (4, NULL, '2026-05-19 15:35:00+00', 'shipped'),     -- 13
    (8, 7,    '2026-06-02 11:15:00+00', 'shipped'),     -- 14
    (3, 8,    '2026-06-20 18:00:00+00', 'pending');     -- 15

INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES
    (1, 1, 1, 1199.00), (1, 4, 1, 29.99),
    (2, 8, 2, 39.00),   (2, 9, 1, 49.00),
    (3, 3, 1, 89.90),
    (4, 2, 1, 999.00),  (4, 5, 1, 45.00),  (4, 4, 2, 29.99),
    (5, 6, 2, 249.00),
    (6, 11, 1, 199.00),
    (7, 10, 1, 55.00),  (7, 8, 1, 39.00),
    (8, 11, 1, 199.00),
    (9, 9, 1, 49.00),   (9, 10, 1, 55.00), (9, 8, 1, 39.00),
    (10, 1, 1, 1299.00), (10, 3, 1, 89.90),
    (11, 4, 3, 29.99),
    (12, 6, 1, 249.00), (12, 5, 1, 45.00),
    (13, 2, 1, 999.00),
    (14, 8, 4, 39.00),  (14, 3, 1, 89.90),
    (15, 1, 1, 1299.00), (15, 11, 1, 199.00);
```

Every statement in that script is explained later: `CREATE TABLE` and identity columns in [Section 35](#35-creating-and-altering-tables), constraints such as `PRIMARY KEY`, `REFERENCES`, and `CHECK` in [Section 36](#36-constraints-and-referential-integrity), and `INSERT` in [Section 32](#32-insert). For now, treat it as the data you will query.

### The data

Here is every table in full, so you can refer back when checking results. Do not worry about the `SELECT` syntax yet.

```sql
SELECT * FROM customers;
```

```text
 customer_id | first_name | last_name |       email        |   city    | country  | signup_date
-------------+------------+-----------+--------------------+-----------+----------+-------------
           1 | Alice      | Martin    | alice@example.com  | London    | UK       | 2025-01-05
           2 | Bruno      | Silva     | bruno@example.com  | Lisbon    | Portugal | 2025-01-18
           3 | Chloe      | Kim       | chloe@example.com  | Toronto   | Canada   | 2025-02-02
           4 | Daniel     | Okafor    | daniel@example.com | Berlin    | Germany  | 2025-02-20
           5 | Emma       | Novak     | emma@example.com   | Toronto   | Canada   | 2025-03-11
           6 | Farid      | Haddad    | farid@example.com  | NULL      | UK       | 2025-03-30
           7 | Grace      | Liu       | grace@example.com  | Boston    | USA      | 2025-04-14
           8 | Hugo       | Berg      | hugo@example.com   | Stockholm | Sweden   | 2025-05-01
           9 | Isla       | Murphy    | isla@example.com   | NULL      | USA      | 2025-05-22
          10 | Jonas      | Weber     | jonas@example.com  | Berlin    | Germany  | 2025-06-09
(10 rows)
```

```sql
SELECT * FROM categories;
```

```text
 category_id |    name     | parent_id
-------------+-------------+-----------
           1 | Electronics |      NULL
           2 | Computers   |         1
           3 | Laptops     |         2
           4 | Accessories |         1
           5 | Books       |      NULL
           6 | Programming |         5
           7 | Home        |      NULL
(7 rows)
```

```sql
SELECT * FROM products;
```

```text
 product_id |            name             | category_id |  price  | stock |                     attributes
------------+-----------------------------+-------------+---------+-------+----------------------------------------------------
          1 | Laptop Pro 14               |           3 | 1299.00 |    15 | {"brand": "Nova", "color": "silver", "ram_gb": 16}
          2 | Laptop Air 13               |           3 |  999.00 |    20 | {"brand": "Nova", "color": "gold", "ram_gb": 8}
          3 | Mechanical Keyboard         |           4 |   89.90 |    50 | {"brand": "Clack", "switches": "brown"}
          4 | Wireless Mouse              |           4 |   29.99 |   120 | {"dpi": 1600, "brand": "Clack"}
          5 | USB-C Hub                   |           4 |   45.00 |     0 | {"brand": "Portly", "ports": 7}
          6 | 27in Monitor                |           2 |  249.00 |    30 | {"brand": "Vista", "size_in": 27}
          7 | Desktop Tower               |           2 | 1499.00 |     5 | {"brand": "Nova", "ram_gb": 32}
          8 | SQL for Everyone            |           6 |   39.00 |   200 | {"pages": 320, "format": "paperback"}
          9 | Python Deep Dive            |           6 |   49.00 |   150 | {"pages": 540, "format": "hardcover"}
         10 | Designing Data Systems      |           6 |   55.00 |    80 | {"pages": 610, "format": "paperback"}
         11 | Noise-Cancelling Headphones |           4 |  199.00 |    25 | {"brand": "Hush"}
         12 | Webcam HD                   |           4 |   59.00 |    40 | NULL
(12 rows)
```

```sql
SELECT * FROM employees;
```

```text
 employee_id | first_name | last_name |      title      | department  | manager_id |  salary   | hire_date
-------------+------------+-----------+-----------------+-------------+------------+-----------+------------
           1 | Sofia      | Reyes     | CEO             | Executive   |       NULL | 180000.00 | 2019-03-01
           2 | Marcus     | Chen      | CTO             | Engineering |          1 | 150000.00 | 2019-06-15
           3 | Priya      | Nair      | Head of Sales   | Sales       |          1 | 120000.00 | 2020-01-10
           4 | Leo        | Novak     | Senior Engineer | Engineering |          2 | 110000.00 | 2020-09-01
           5 | Mia        | Laine     | Engineer        | Engineering |          2 |  95000.00 | 2022-02-14
           6 | Omar       | Farah     | Engineer        | Engineering |          4 |  95000.00 | 2023-05-02
           7 | Nina       | Park      | Sales Rep       | Sales       |          3 |  60000.00 | 2021-07-19
           8 | Tom        | Walsh     | Sales Rep       | Sales       |          3 |  60000.00 | 2024-01-08
           9 | Eva        | Brandt    | Support Lead    | Support     |          1 |  70000.00 | 2021-11-03
(9 rows)
```

```sql
SELECT * FROM orders;
```

```text
 order_id | customer_id | employee_id |       ordered_at       |  status
----------+-------------+-------------+------------------------+-----------
        1 |           1 |           7 | 2026-01-05 09:15:00+00 | delivered
        2 |           2 |           8 | 2026-01-12 14:30:00+00 | delivered
        3 |           1 |           7 | 2026-01-28 19:45:00+00 | delivered
        4 |           3 |           7 | 2026-02-03 11:00:00+00 | delivered
        5 |           4 |           8 | 2026-02-14 16:20:00+00 | delivered
        6 |           5 |        NULL | 2026-02-27 08:05:00+00 | cancelled
        7 |           3 |           7 | 2026-03-09 13:10:00+00 | delivered
        8 |           6 |           8 | 2026-03-15 10:40:00+00 | delivered
        9 |           1 |           8 | 2026-03-30 21:05:00+00 | delivered
       10 |           7 |           7 | 2026-04-11 12:00:00+00 | delivered
       11 |           2 |           7 | 2026-04-25 17:25:00+00 | delivered
       12 |           5 |           8 | 2026-05-06 09:50:00+00 | delivered
       13 |           4 |        NULL | 2026-05-19 15:35:00+00 | shipped
       14 |           8 |           7 | 2026-06-02 11:15:00+00 | shipped
       15 |           3 |           8 | 2026-06-20 18:00:00+00 | pending
(15 rows)
```

```sql
SELECT * FROM order_items;
```

```text
 order_id | product_id | quantity | unit_price
----------+------------+----------+------------
        1 |          1 |        1 |    1199.00
        1 |          4 |        1 |      29.99
        2 |          8 |        2 |      39.00
        2 |          9 |        1 |      49.00
        3 |          3 |        1 |      89.90
        4 |          2 |        1 |     999.00
        4 |          5 |        1 |      45.00
        4 |          4 |        2 |      29.99
        5 |          6 |        2 |     249.00
        6 |         11 |        1 |     199.00
        7 |         10 |        1 |      55.00
        7 |          8 |        1 |      39.00
        8 |         11 |        1 |     199.00
        9 |          9 |        1 |      49.00
        9 |         10 |        1 |      55.00
        9 |          8 |        1 |      39.00
       10 |          1 |        1 |    1299.00
       10 |          3 |        1 |      89.90
       11 |          4 |        3 |      29.99
       12 |          6 |        1 |     249.00
       12 |          5 |        1 |      45.00
       13 |          2 |        1 |     999.00
       14 |          8 |        4 |      39.00
       14 |          3 |        1 |      89.90
       15 |          1 |        1 |    1299.00
       15 |         11 |        1 |     199.00
(26 rows)
```

`psql` can also describe a table's structure, which is the fastest way to learn an unfamiliar schema:

```sql
\d orders
```

```text
                                      Table "public.orders"
   Column    |           Type           | Collation | Nullable |             Default
-------------+--------------------------+-----------+----------+----------------------------------
 order_id    | integer                  |           | not null | generated by default as identity
 customer_id | integer                  |           | not null |
 employee_id | integer                  |           |          |
 ordered_at  | timestamp with time zone |           | not null |
 status      | text                     |           | not null |
Indexes:
    "orders_pkey" PRIMARY KEY, btree (order_id)
Check constraints:
    "orders_status_check" CHECK (status = ANY (ARRAY['pending'::text, 'shipped'::text, 'delivered'::text, 'cancelled'::text]))
Foreign-key constraints:
    "orders_customer_id_fkey" FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
    "orders_employee_id_fkey" FOREIGN KEY (employee_id) REFERENCES employees(employee_id)
Referenced by:
    TABLE "order_items" CONSTRAINT "order_items_order_id_fkey" FOREIGN KEY (order_id) REFERENCES orders(order_id)
```

**Notes:**
- `NULL` in these outputs is how this guide displays missing values (set with `\pset null NULL` in `psql`). By default `psql` shows them as blank, which is easy to confuse with an empty string.
- `ordered_at` is a `timestamptz` (timestamp with time zone) and the session time zone is UTC, hence the `+00` suffix. [Section 14](#14-dates-times-and-intervals) explains why this is the right type for event times.
- `order_items.unit_price` deliberately differs from `products.price` for order 1: the Laptop Pro 14 cost 1,199.00 in January and 1,299.00 later. Storing the price at the time of sale is a real-world design rule, covered in [Section 37](#37-normalization-and-schema-design).
- Every section of this guide starts from this exact data. When a section changes rows, the next section starts fresh.

---

## 5. How a Query Runs: Written Order vs Logical Order

A `SELECT` statement is written in one order but evaluated in another. This is the single most useful fact for understanding SQL errors, and a favorite interview question.

```text
Written order                      Logical evaluation order
─────────────                      ────────────────────────
SELECT     columns, expressions    1. FROM / JOIN   pick the source tables and combine them
FROM       tables                  2. WHERE         keep only rows that match (row by row)
JOIN       more tables             3. GROUP BY      collapse rows into groups
WHERE      row filter              4. HAVING        keep only groups that match
GROUP BY   grouping columns        5. SELECT        compute output columns (and window functions)
HAVING     group filter            6. DISTINCT      remove duplicate output rows
ORDER BY   sort keys               7. ORDER BY      sort the result
LIMIT      row count               8. LIMIT/OFFSET  cut the result down
```

Each step receives the table produced by the previous step. Several rules fall straight out of this order:

**Aliases defined in `SELECT` do not exist yet in `WHERE`**, because `WHERE` runs before `SELECT`:

```sql
SELECT name, price * 1.24 AS price_with_vat
FROM products
WHERE price_with_vat > 100;
```

```text
ERROR:  column "price_with_vat" does not exist
LINE 3: WHERE price_with_vat > 100;
              ^
```

Repeat the expression instead, or wrap the query so the alias exists by the time you filter ([Section 24](#24-subqueries)):

```sql
SELECT name, price * 1.24 AS price_with_vat
FROM products
WHERE price * 1.24 > 1000;
```

```text
     name      | price_with_vat
---------------+----------------
 Laptop Pro 14 |      1610.7600
 Laptop Air 13 |      1238.7600
 Desktop Tower |      1858.7600
(3 rows)
```

**`ORDER BY` runs after `SELECT`, so it can use the alias:**

```sql
SELECT name, price * 1.24 AS price_with_vat
FROM products
ORDER BY price_with_vat DESC
LIMIT 3;
```

```text
     name      | price_with_vat
---------------+----------------
 Desktop Tower |      1858.7600
 Laptop Pro 14 |      1610.7600
 Laptop Air 13 |      1238.7600
(3 rows)
```

**Aggregates cannot appear in `WHERE`**, because `WHERE` filters individual rows before any grouping has happened. Group-level conditions belong in `HAVING`:

```sql
SELECT customer_id, count(*) AS orders
FROM orders
WHERE count(*) > 2
GROUP BY customer_id;
```

```text
ERROR:  aggregate functions are not allowed in WHERE
LINE 3: WHERE count(*) > 2
              ^
```

```sql
SELECT customer_id, count(*) AS orders
FROM orders
GROUP BY customer_id
HAVING count(*) > 2;
```

```text
 customer_id | orders
-------------+--------
           3 |      3
           1 |      3
(2 rows)
```

**`LIMIT` without `ORDER BY` returns arbitrary rows.** `LIMIT` runs last and simply takes the first rows of whatever order the result happens to be in. Without `ORDER BY`, that order is whatever was cheapest for the database, and it can change between runs.

This order is *logical*, not physical. The query planner is free to execute things differently — pushing filters down, using an index to avoid sorting — as long as the result is the same as if the steps ran in this order. [Section 42](#42-query-plans-and-performance) shows how to see the physical plan.

**Notes:**
- Memorize the order: FROM, WHERE, GROUP BY, HAVING, SELECT, DISTINCT, ORDER BY, LIMIT. A mnemonic is "**F**rank **W**ent **G**rocery shopping, **H**e **S**aw **D**onuts, **O**rdered **L**ots".
- PostgreSQL allows `GROUP BY` and `ORDER BY` to refer to output column aliases or positions (`GROUP BY 1`) as a convenience. `WHERE` and `HAVING` never can. MySQL and SQLite are more lenient and allow aliases in `HAVING`, which is non-standard.
- Window functions are computed in step 5, after `WHERE` and `HAVING`. That is why you cannot filter on a window function's result without wrapping the query ([Section 27](#27-window-function-basics)).
- Interview phrasing: "What is the difference between `WHERE` and `HAVING`?" `WHERE` filters rows before grouping; `HAVING` filters groups after grouping, and can therefore use aggregates.

---

## 6. SELECT, FROM, and Aliases

`SELECT` chooses **which columns** to return and `FROM` chooses **which table** to read. The simplest useful query names the columns you want:

```sql
SELECT first_name, last_name, country
FROM customers;
```

```text
 first_name | last_name | country
------------+-----------+----------
 Alice      | Martin    | UK
 Bruno      | Silva     | Portugal
 Chloe      | Kim       | Canada
 Daniel     | Okafor    | Germany
 Emma       | Novak     | Canada
 Farid      | Haddad    | UK
 Grace      | Liu       | USA
 Hugo       | Berg      | Sweden
 Isla       | Murphy    | USA
 Jonas      | Weber     | Germany
(10 rows)
```

`SELECT *` returns every column. It is perfect for exploring a table interactively — and wrong in application code and saved queries, because it silently changes shape when someone adds a column, moves more data than needed, and hides which columns the code depends on.

### Expressions in SELECT

A select list can contain any expression, not just column names: arithmetic, function calls, constants, and combinations:

```sql
SELECT name,
       price,
       stock,
       price * stock        AS stock_value,
       upper(name)          AS shouting,
       'EUR'                AS currency
FROM products
WHERE category_id = 3;
```

```text
     name      |  price  | stock | stock_value |   shouting    | currency
---------------+---------+-------+-------------+---------------+----------
 Laptop Pro 14 | 1299.00 |    15 |    19485.00 | LAPTOP PRO 14 | EUR
 Laptop Air 13 |  999.00 |    20 |    19980.00 | LAPTOP AIR 13 | EUR
(2 rows)
```

### Column aliases

`AS` gives an output column a name. Without an alias, PostgreSQL invents one (`?column?` for arithmetic, the function name for function calls). The `AS` keyword is optional (`price * stock stock_value` also works), but writing it makes the intent obvious. An alias containing spaces or capitals needs double quotes:

```sql
SELECT first_name || ' ' || last_name AS "Full Name",
       signup_date                     AS joined
FROM customers
LIMIT 3;
```

```text
  Full Name   |   joined
--------------+------------
 Alice Martin | 2025-01-05
 Bruno Silva  | 2025-01-18
 Chloe Kim    | 2025-02-02
(3 rows)
```

### Table aliases

A table alias is a short name for a table within one query. It is optional for single-table queries and essential once you join tables ([Section 20](#20-inner-join)), where it tells the reader (and the database) which table each column comes from:

```sql
SELECT c.first_name, c.city
FROM customers AS c
WHERE c.country = 'Canada';
```

```text
 first_name |  city
------------+---------
 Chloe      | Toronto
 Emma       | Toronto
(2 rows)
```

### SELECT without FROM

PostgreSQL lets you evaluate expressions with no table at all, which makes `psql` a handy calculator and function tester:

```sql
SELECT 2 + 3 * 4 AS result, 7 / 2 AS int_division, 7.0 / 2 AS decimal_division, upper('sql') AS shout;
```

```text
 result | int_division |  decimal_division  | shout
--------+--------------+--------------------+-------
     14 |            3 | 3.5000000000000000 | SQL
(1 row)
```

**Notes:**
- Integer divided by integer is integer division in PostgreSQL, SQL Server, and SQLite: `7 / 2` is `3`. Make one side decimal (`7.0 / 2` or `7::numeric / 2`) to get `3.5`. MySQL returns `3.5` either way.
- Oracle requires a `FROM` clause, conventionally the dummy table `DUAL` (`SELECT 1 FROM dual`) — recent Oracle versions relaxed this. PostgreSQL, MySQL, SQLite, and SQL Server do not need it.
- A column alias cannot be used in the same `SELECT` list that defines it: `SELECT price * 2 AS double, double + 1` fails, because all expressions in one select list are computed "at the same time".
- Prefer explicit column lists over `SELECT *` in anything that is saved, shared, or run by code.

---

## 7. Filtering with WHERE

`WHERE` keeps only the rows for which a condition is **true**. The condition is evaluated once per row.

### Comparison operators

| Operator | Meaning |
|:--|:--|
| `=` | Equal (a single `=`, not `==`) |
| `<>` or `!=` | Not equal (`<>` is the standard spelling) |
| `<`, `>`, `<=`, `>=` | Ordering comparisons (work on numbers, text, dates) |

```sql
SELECT name, price
FROM products
WHERE price >= 200;
```

```text
     name      |  price
---------------+---------
 Laptop Pro 14 | 1299.00
 Laptop Air 13 |  999.00
 27in Monitor  |  249.00
 Desktop Tower | 1499.00
(4 rows)
```

Text comparisons are case-sensitive in PostgreSQL, and dates compare chronologically:

```sql
SELECT first_name, signup_date
FROM customers
WHERE signup_date < '2025-03-01';
```

```text
 first_name | signup_date
------------+-------------
 Alice      | 2025-01-05
 Bruno      | 2025-01-18
 Chloe      | 2025-02-02
 Daniel     | 2025-02-20
(4 rows)
```

### Combining conditions: AND, OR, NOT

`AND` requires both sides to be true; `OR` requires at least one; `NOT` inverts. **`AND` binds tighter than `OR`**, exactly like multiplication binds tighter than addition, and this is a classic source of wrong results:

```sql
-- Intended: books OR accessories, but only the cheap ones. Missing parentheses:
SELECT name, category_id, price
FROM products
WHERE category_id = 6 OR category_id = 4 AND price < 50;
```

```text
          name          | category_id | price
------------------------+-------------+-------
 Wireless Mouse         |           4 | 29.99
 USB-C Hub              |           4 | 45.00
 SQL for Everyone       |           6 | 39.00
 Python Deep Dive       |           6 | 49.00
 Designing Data Systems |           6 | 55.00
(5 rows)
```

That query returned the 55.00 book because it was read as `category_id = 6 OR (category_id = 4 AND price < 50)`. Parentheses make the intent explicit:

```sql
SELECT name, category_id, price
FROM products
WHERE (category_id = 6 OR category_id = 4) AND price < 50;
```

```text
       name       | category_id | price
------------------+-------------+-------
 Wireless Mouse   |           4 | 29.99
 USB-C Hub        |           4 | 45.00
 SQL for Everyone |           6 | 39.00
 Python Deep Dive |           6 | 49.00
(4 rows)
```

### IN and NOT IN

`IN` tests membership in a list and replaces a chain of `OR` equalities:

```sql
SELECT order_id, status
FROM orders
WHERE status IN ('pending', 'shipped');
```

```text
 order_id | status
----------+---------
       13 | shipped
       14 | shipped
       15 | pending
(3 rows)
```

### BETWEEN

`BETWEEN a AND b` is shorthand for `>= a AND <= b`. **Both ends are inclusive**, which matters for dates:

```sql
SELECT name, price
FROM products
WHERE price BETWEEN 45 AND 89.90;
```

```text
          name          | price
------------------------+-------
 Mechanical Keyboard    | 89.90
 USB-C Hub              | 45.00
 Python Deep Dive       | 49.00
 Designing Data Systems | 55.00
 Webcam HD              | 59.00
(5 rows)
```

For timestamps, `BETWEEN '2026-01-01' AND '2026-01-31'` silently excludes everything after midnight on the 31st, because `'2026-01-31'` means `2026-01-31 00:00:00`. Use a half-open range instead, which is correct for every type and every month length:

```sql
SELECT order_id, ordered_at
FROM orders
WHERE ordered_at >= '2026-01-01' AND ordered_at < '2026-02-01';
```

```text
 order_id |       ordered_at
----------+------------------------
        1 | 2026-01-05 09:15:00+00
        2 | 2026-01-12 14:30:00+00
        3 | 2026-01-28 19:45:00+00
(3 rows)
```

### LIKE and ILIKE: simple patterns

`LIKE` matches text against a pattern with two wildcards: `%` matches any sequence of characters (including none) and `_` matches exactly one character. `ILIKE` is PostgreSQL's case-insensitive version.

```sql
SELECT name FROM products WHERE name LIKE 'Laptop%';        -- starts with
```

```text
     name
---------------
 Laptop Pro 14
 Laptop Air 13
(2 rows)
```

```sql
SELECT name FROM products WHERE name ILIKE '%data%';        -- contains, any case
```

```text
          name
------------------------
 Designing Data Systems
(1 row)
```

```sql
SELECT email FROM customers WHERE email LIKE '_____@%';     -- exactly five characters before the @
```

```text
       email
-------------------
 alice@example.com
 bruno@example.com
 chloe@example.com
 farid@example.com
 grace@example.com
 jonas@example.com
(6 rows)
```

To match a literal `%` or `_`, escape it: `LIKE '50\%%'` matches text starting with `50%`. Regular expressions, for anything more complex, are in [Section 12](#12-strings-and-pattern-matching).

**Notes:**
- Use single quotes for strings: `WHERE country = 'UK'`. `WHERE country = "UK"` looks for a *column* named UK and fails.
- When a `WHERE` clause mixes `AND` and `OR`, always add parentheses, even when precedence happens to give the right answer. The next reader should not have to work it out.
- A leading wildcard (`LIKE '%term'`) cannot use a normal B-tree index, so it scans the whole table. That is fine for 12 rows and slow for 12 million ([Section 39](#39-indexes)).
- MySQL's default collations are case-insensitive, so `LIKE` there behaves like PostgreSQL's `ILIKE`. SQL Server's default is also case-insensitive. PostgreSQL and SQLite (for non-ASCII) are case-sensitive.
- `NOT IN` has a dangerous interaction with `NULL`, covered in the next section.

---

## 8. NULL and Three-Valued Logic

`NULL` means **unknown or missing**. It is not zero, not an empty string, and not false. It is the absence of a value — and understanding how it behaves separates people who write SQL from people who write *correct* SQL.

### Comparisons with NULL are never true

Any comparison involving `NULL` produces `NULL` (unknown), not true or false. `NULL = NULL` is not true, because two unknown values are not known to be equal:

```sql
SELECT NULL = NULL      AS eq,
       NULL <> NULL     AS neq,
       NULL = 1         AS eq_one,
       NULL + 1         AS plus_one,
       'a' || NULL      AS concat;
```

```text
  eq  | neq  | eq_one | plus_one | concat
------+------+--------+----------+--------
 NULL | NULL | NULL   |     NULL | NULL
(1 row)
```

Since `WHERE` keeps only rows where the condition is *true*, a row whose condition evaluates to `NULL` is dropped. So this finds nothing — even though two customers have no city:

```sql
SELECT first_name, city FROM customers WHERE city = NULL;
```

```text
 first_name | city
------------+------
(0 rows)
```

### IS NULL and IS NOT NULL

The only correct way to test for `NULL` is with `IS NULL` or `IS NOT NULL`:

```sql
SELECT first_name, city FROM customers WHERE city IS NULL;
```

```text
 first_name | city
------------+------
 Farid      | NULL
 Isla       | NULL
(2 rows)
```

### Three-valued logic

Because conditions can be true, false, or unknown, SQL's boolean logic has three values. The rules follow from asking "could the unknown value change the answer?":

| `a` | `b` | `a AND b` | `a OR b` | `NOT a` |
|:--|:--|:--|:--|:--|
| true | NULL | NULL | true | false |
| false | NULL | false | NULL | true |
| NULL | NULL | NULL | NULL | NULL |

`false AND anything` is false, and `true OR anything` is true, regardless of the unknown.

### The silent row loss

Filters with `<>` quietly exclude `NULL` rows. Asking for "customers not in London" misses the two customers whose city is unknown:

```sql
SELECT first_name, city FROM customers WHERE city <> 'London';
```

```text
 first_name |   city
------------+-----------
 Bruno      | Lisbon
 Chloe      | Toronto
 Daniel     | Berlin
 Emma       | Toronto
 Grace      | Boston
 Hugo       | Stockholm
 Jonas      | Berlin
(7 rows)
```

If unknown cities should count as "not London", say so explicitly:

```sql
SELECT first_name, city FROM customers WHERE city <> 'London' OR city IS NULL;
```

```text
 first_name |   city
------------+-----------
 Bruno      | Lisbon
 Chloe      | Toronto
 Daniel     | Berlin
 Emma       | Toronto
 Farid      | NULL
 Grace      | Boston
 Hugo       | Stockholm
 Isla       | NULL
 Jonas      | Berlin
(9 rows)
```

PostgreSQL (and the SQL standard) also offer a null-safe comparison, `IS DISTINCT FROM`, which treats two `NULL`s as equal and a `NULL` versus a value as different:

```sql
SELECT first_name, city FROM customers WHERE city IS DISTINCT FROM 'London';
```

```text
 first_name |   city
------------+-----------
 Bruno      | Lisbon
 Chloe      | Toronto
 Daniel     | Berlin
 Emma       | Toronto
 Farid      | NULL
 Grace      | Boston
 Hugo       | Stockholm
 Isla       | NULL
 Jonas      | Berlin
(9 rows)
```

### The NOT IN trap

`x NOT IN (a, b, NULL)` expands to `x <> a AND x <> b AND x <> NULL`. The last comparison is always `NULL`, so the whole condition can never be true, and the query returns **no rows at all**:

```sql
SELECT first_name FROM customers WHERE city NOT IN ('London', NULL);
```

```text
 first_name
------------
(0 rows)
```

This bites hardest when the list comes from a subquery on a nullable column (`WHERE id NOT IN (SELECT manager_id FROM employees)`), because the `NULL` is invisible. [Section 22](#22-self-cross-semi-anti-and-lateral-joins) shows the safe alternative, `NOT EXISTS`.

### NULL in other clauses

- **Aggregates ignore NULL**: `count(city)` counts non-null cities, `avg(x)` averages only known values ([Section 16](#16-aggregate-functions)).
- **`GROUP BY` puts all NULLs in one group**, and `DISTINCT` treats them as one value.
- **Sorting**: PostgreSQL sorts `NULL` as larger than any value (last in `ASC`, first in `DESC`), controllable with `NULLS FIRST` and `NULLS LAST` ([Section 9](#9-order-by-limit-and-distinct)).
- **`UNIQUE` constraints** allow multiple `NULL`s by default, since unknowns are not equal to each other.

**Notes:**
- Never write `= NULL` or `<> NULL`. They are always unknown. Use `IS NULL`, `IS NOT NULL`, or `IS [NOT] DISTINCT FROM`.
- Any arithmetic or concatenation with `NULL` yields `NULL`. Use `COALESCE` to supply a default ([Section 15](#15-coalesce-nullif-greatest-least)).
- "What does `NULL = NULL` return?" is a staple interview question. The answer is `NULL` (unknown), which `WHERE` treats as not true.
- Decide deliberately whether a column may be `NULL`. Declaring columns `NOT NULL` wherever a value is always required removes a whole category of bugs ([Section 36](#36-constraints-and-referential-integrity)).

---

## 9. ORDER BY, LIMIT, and DISTINCT

### ORDER BY

Rows in a table have no inherent order, and neither does a query result unless you ask for one. `ORDER BY` sorts by one or more expressions, ascending (`ASC`, the default) or descending (`DESC`). Later keys break ties in earlier ones:

```sql
SELECT first_name, last_name, department, salary
FROM employees
ORDER BY department ASC, salary DESC, last_name;
```

```text
 first_name | last_name | department  |  salary
------------+-----------+-------------+-----------
 Marcus     | Chen      | Engineering | 150000.00
 Leo        | Novak     | Engineering | 110000.00
 Omar       | Farah     | Engineering |  95000.00
 Mia        | Laine     | Engineering |  95000.00
 Sofia      | Reyes     | Executive   | 180000.00
 Priya      | Nair      | Sales       | 120000.00
 Nina       | Park      | Sales       |  60000.00
 Tom        | Walsh     | Sales       |  60000.00
 Eva        | Brandt    | Support     |  70000.00
(9 rows)
```

You can sort by expressions and by aliases, and control where `NULL`s go:

```sql
SELECT first_name, city
FROM customers
ORDER BY city NULLS FIRST;
```

```text
 first_name |   city
------------+-----------
 Farid      | NULL
 Isla       | NULL
 Jonas      | Berlin
 Daniel     | Berlin
 Grace      | Boston
 Bruno      | Lisbon
 Alice      | London
 Hugo       | Stockholm
 Emma       | Toronto
 Chloe      | Toronto
(10 rows)
```

### LIMIT and OFFSET

`LIMIT n` returns at most `n` rows; `OFFSET k` skips the first `k`. Combined with `ORDER BY`, they give "top N" queries and simple pagination:

```sql
SELECT name, price
FROM products
ORDER BY price DESC
LIMIT 3;
```

```text
     name      |  price
---------------+---------
 Desktop Tower | 1499.00
 Laptop Pro 14 | 1299.00
 Laptop Air 13 |  999.00
(3 rows)
```

```sql
-- "Page 2" with 4 products per page
SELECT product_id, name
FROM products
ORDER BY product_id
LIMIT 4 OFFSET 4;
```

```text
 product_id |       name
------------+------------------
          5 | USB-C Hub
          6 | 27in Monitor
          7 | Desktop Tower
          8 | SQL for Everyone
(4 rows)
```

The SQL-standard spelling, supported by PostgreSQL, Oracle, and SQL Server, is `OFFSET 4 ROWS FETCH FIRST 4 ROWS ONLY`. Its `WITH TIES` variant also returns any rows tied with the last one, which matters when ties are meaningful:

```sql
SELECT first_name, salary
FROM employees
ORDER BY salary
FETCH FIRST 1 ROWS WITH TIES;
```

```text
 first_name |  salary
------------+----------
 Nina       | 60000.00
 Tom        | 60000.00
(2 rows)
```

`LIMIT 1` would have returned one of the two 60,000 earners, chosen arbitrarily — `WITH TIES` returns both.

### Why large OFFSETs are slow

`OFFSET 100000` still reads and discards 100,000 rows. For deep pagination, use **keyset pagination** (also called seek pagination): remember the last key you showed and ask for rows after it. With an index on the key, this is fast at any depth:

```sql
-- The previous page ended at product_id 4
SELECT product_id, name
FROM products
WHERE product_id > 4
ORDER BY product_id
LIMIT 4;
```

```text
 product_id |       name
------------+------------------
          5 | USB-C Hub
          6 | 27in Monitor
          7 | Desktop Tower
          8 | SQL for Everyone
(4 rows)
```

### DISTINCT

`DISTINCT` removes duplicate rows from the result. It applies to the **entire row** of selected columns, not to the first column:

```sql
SELECT DISTINCT country FROM customers ORDER BY country;
```

```text
 country
----------
 Canada
 Germany
 Portugal
 Sweden
 UK
 USA
(6 rows)
```

```sql
SELECT DISTINCT country, city FROM customers ORDER BY country, city;
```

```text
 country  |   city
----------+-----------
 Canada   | Toronto
 Germany  | Berlin
 Portugal | Lisbon
 Sweden   | Stockholm
 UK       | London
 UK       | NULL
 USA      | Boston
 USA      | NULL
(8 rows)
```

PostgreSQL adds `DISTINCT ON (expr)`, which keeps the **first row of each group** according to `ORDER BY`. It is the shortest way to answer "the most recent order per customer" or "the most expensive product per category":

```sql
SELECT DISTINCT ON (category_id) category_id, name, price
FROM products
ORDER BY category_id, price DESC;
```

```text
 category_id |            name             |  price
-------------+-----------------------------+---------
           2 | Desktop Tower               | 1499.00
           3 | Laptop Pro 14               | 1299.00
           4 | Noise-Cancelling Headphones |  199.00
           6 | Designing Data Systems      |   55.00
(4 rows)
```

The portable way to do the same thing, which works in every database, is with a window function ([Section 30](#30-classic-patterns-top-n-dedup-gaps-and-islands-pivots)).

**Notes:**
- Without `ORDER BY`, result order is undefined. It may look stable in testing and change after a data update, a new index, or a version upgrade.
- `ORDER BY` a non-unique column gives an undefined order among ties. For stable pagination, always add a unique tiebreaker (`ORDER BY price DESC, product_id`).
- `LIMIT` is PostgreSQL, MySQL, and SQLite. SQL Server uses `SELECT TOP 3 ...` or `OFFSET ... FETCH`. Oracle uses `FETCH FIRST` (or the old `ROWNUM`).
- `DISTINCT` is often a symptom of a join that multiplies rows unexpectedly. If you reach for it to "fix" duplicates, first find out where they come from ([Section 20](#20-inner-join)).
- In `DISTINCT ON`, the `ORDER BY` must start with the same expressions as the `DISTINCT ON` list.

---

## 10. Expressions and CASE

`CASE` is SQL's if-else. It is an **expression**, so it can appear anywhere a value can — in `SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`, inside aggregates, and inside other expressions.

### Searched CASE

The general form tests conditions in order and returns the result of the **first** true one. If none match, it returns the `ELSE` value, or `NULL` if there is no `ELSE`:

```sql
SELECT name,
       price,
       CASE
           WHEN price >= 1000 THEN 'premium'
           WHEN price >= 100  THEN 'mid-range'
           ELSE 'budget'
       END AS tier
FROM products
ORDER BY price DESC;
```

```text
            name             |  price  |   tier
-----------------------------+---------+-----------
 Desktop Tower               | 1499.00 | premium
 Laptop Pro 14               | 1299.00 | premium
 Laptop Air 13               |  999.00 | mid-range
 27in Monitor                |  249.00 | mid-range
 Noise-Cancelling Headphones |  199.00 | mid-range
 Mechanical Keyboard         |   89.90 | budget
 Webcam HD                   |   59.00 | budget
 Designing Data Systems      |   55.00 | budget
 Python Deep Dive            |   49.00 | budget
 USB-C Hub                   |   45.00 | budget
 SQL for Everyone            |   39.00 | budget
 Wireless Mouse              |   29.99 | budget
(12 rows)
```

Order matters — a price of 1,299 also satisfies `price >= 100`, but the first matching branch wins.

### Simple CASE

When every branch compares the same expression for equality, the shorter form reads better:

```sql
SELECT order_id,
       status,
       CASE status
           WHEN 'pending'   THEN 1
           WHEN 'shipped'   THEN 2
           WHEN 'delivered' THEN 3
           ELSE 0
       END AS stage
FROM orders
WHERE order_id IN (6, 13, 14, 15);
```

```text
 order_id |  status   | stage
----------+-----------+-------
        6 | cancelled |     0
       13 | shipped   |     2
       14 | shipped   |     2
       15 | pending   |     1
(4 rows)
```

### CASE in ORDER BY: custom sort orders

Sorting text alphabetically rarely matches business meaning. `CASE` imposes any order you like:

```sql
SELECT order_id, status
FROM orders
WHERE order_id BETWEEN 11 AND 15
ORDER BY CASE status WHEN 'pending' THEN 1 WHEN 'shipped' THEN 2 ELSE 3 END, order_id;
```

```text
 order_id |  status
----------+-----------
       15 | pending
       13 | shipped
       14 | shipped
       11 | delivered
       12 | delivered
(5 rows)
```

### CASE inside aggregates: conditional counting

Putting `CASE` inside `sum` or `count` computes several filtered totals in a single pass over the table. This pattern (sometimes called *conditional aggregation*) is the basis of pivot tables ([Section 30](#30-classic-patterns-top-n-dedup-gaps-and-islands-pivots)):

```sql
SELECT count(*)                                              AS total_orders,
       sum(CASE WHEN status = 'delivered' THEN 1 ELSE 0 END) AS delivered,
       sum(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END) AS cancelled,
       count(CASE WHEN employee_id IS NULL THEN 1 END)       AS no_sales_rep
FROM orders;
```

```text
 total_orders | delivered | cancelled | no_sales_rep
--------------+-----------+-----------+--------------
           15 |        11 |         1 |            2
(1 row)
```

The last column relies on `count` ignoring `NULL` — the `CASE` without `ELSE` returns `NULL` for non-matching rows, so only matches are counted. PostgreSQL has a cleaner syntax for this, `FILTER`, covered in [Section 18](#18-filter-rollup-cube-and-grouping-sets).

**Notes:**
- All `THEN` branches must return compatible types. `CASE WHEN x THEN 1 ELSE 'none' END` fails, since `'none'` is not an integer.
- A `CASE` without `ELSE` returns `NULL` when nothing matches. This is sometimes intended (conditional counting) and sometimes a bug (a category silently becoming `NULL`).
- Simple `CASE x WHEN NULL THEN ...` never matches, because it tests `x = NULL`. Use the searched form, `CASE WHEN x IS NULL THEN ...`.
- MySQL has `IF(cond, a, b)` and SQL Server has `IIF(cond, a, b)` as shortcuts. `CASE` is standard and works everywhere.

---

## 11. Data Types and Casting

Every column has a **type**, and the type decides which values are allowed, how much space they take, how they sort, and which operators and functions apply. Choosing types well is the first line of defense against bad data — a `date` column simply cannot hold `'next Tuesday'`.

### The types you will actually use

| Category | Type | Holds | Use for |
|:--|:--|:--|:--|
| Integer | `smallint`, `integer`, `bigint` | 2, 4, 8-byte whole numbers | Counts, quantities, IDs (`bigint` for IDs in big tables) |
| Exact decimal | `numeric(p, s)` | Exact decimals, `p` digits total, `s` after the point | **Money** and anything that must add up exactly |
| Floating point | `real`, `double precision` | Approximate binary floats | Scientific measurements, ML features, where tiny errors are acceptable |
| Text | `text`, `varchar(n)` | Strings, optionally with a max length | Names, emails, descriptions (`text` is the PostgreSQL default) |
| Boolean | `boolean` | `true`, `false`, `NULL` | Flags |
| Date and time | `date`, `timestamp`, `timestamptz`, `interval`, `time` | Calendar dates, points in time, durations | See [Section 14](#14-dates-times-and-intervals) |
| Identifier | `uuid` | 128-bit unique IDs | Public IDs, IDs generated outside the database |
| Semi-structured | `jsonb`, arrays like `text[]` | Documents and lists | Flexible attributes ([Section 40](#40-json-arrays-and-full-text-search)) |
| Binary | `bytea` | Raw bytes | Small files, hashes (large files belong in object storage) |

### Why money is never a float

Floating-point types store binary approximations. They are fast and fine for measurements, but they cannot represent most decimal fractions exactly, so sums drift:

```sql
SELECT 0.1::double precision + 0.2::double precision AS float_sum,
       0.1::numeric + 0.2::numeric                   AS numeric_sum,
       (0.1::double precision + 0.2::double precision) = 0.3 AS float_equal;
```

```text
      float_sum      | numeric_sum | float_equal
---------------------+-------------+-------------
 0.30000000000000004 |         0.3 | f
(1 row)
```

`numeric(10,2)`, used for every price in the sample database, stores exactly two decimal places and never drifts.

### text vs varchar(n) vs char(n)

In PostgreSQL, `text` and `varchar` are the same type internally with identical performance — `varchar(n)` merely adds a length check. `char(n)` pads with spaces to exactly `n` characters and is almost never what you want. Other databases differ: SQL Server and MySQL traditionally require `varchar(n)` with a length, and SQL Server uses `nvarchar` for Unicode.

### Casting between types

Conversion is done with the standard `CAST(value AS type)` or PostgreSQL's shorthand `value::type`. Both are identical:

```sql
SELECT CAST('42' AS integer)       AS standard_cast,
       '42'::integer + 1            AS shorthand_cast,
       '2026-03-15'::date + 10      AS date_plus_days,
       3.99::integer                AS rounds_not_truncates,
       'true'::boolean              AS bool;
```

```text
 standard_cast | shorthand_cast | date_plus_days | rounds_not_truncates | bool
---------------+----------------+----------------+----------------------+------
            42 |             43 | 2026-03-25     |                    4 | t
(1 row)
```

A cast that cannot succeed is an error, not a silent `NULL`:

```sql
SELECT 'twelve'::integer;
```

```text
ERROR:  invalid input syntax for type integer: "twelve"
LINE 1: SELECT 'twelve'::integer;
               ^
```

`pg_typeof` shows the type of any expression, which is handy when an operator refuses to work:

```sql
SELECT pg_typeof(1) AS a, pg_typeof(1.5) AS b, pg_typeof('x') AS c, pg_typeof(now()) AS d;
```

```text
    a    |    b    |    c    |            d
---------+---------+---------+--------------------------
 integer | numeric | unknown | timestamp with time zone
(1 row)
```

PostgreSQL is strict about implicit conversion. Comparing a number to a string that looks like a number works because the string literal has no type yet (`price > '100'`), but comparing a number to a **text column** fails. This strictness is a feature — it catches bugs that MySQL and SQLite silently paper over by converting `'abc'` to `0`.

**Notes:**
- Default choices that are right 95 percent of the time: `bigint` or `integer` identity for keys, `numeric` for money, `text` for strings, `timestamptz` for moments in time, `boolean` for flags.
- `numeric` without precision (`numeric` rather than `numeric(10,2)`) stores any number of digits exactly. It is slower than integers but still exact.
- Integer overflow is an error, not a wraparound: `SELECT 2147483647 + 1` fails for `integer`. IDs in large tables should be `bigint` from day one, because changing a key's type later on a big table is painful.
- SQL Server has no `boolean` type (it uses `bit`), and Oracle added one only recently. Portable code sometimes uses `smallint` 0/1 for flags.
- `::type` is PostgreSQL syntax (DuckDB and Redshift copy it). `CAST(... AS ...)` works everywhere.

---

## 12. Strings and Pattern Matching

Text cleaning is a large share of real SQL work — fixing inconsistent capitalization, trimming stray spaces, extracting parts of codes, and matching patterns. These are the functions you will reach for most.

### Everyday string functions

```sql
SELECT name,
       length(name)                    AS len,
       upper(name)                     AS up,
       lower(name)                     AS low,
       left(name, 6)                   AS first6,
       right(name, 3)                  AS last3,
       substring(name FROM 8 FOR 3)    AS mid,
       position('o' IN name)           AS first_o
FROM products
WHERE product_id IN (1, 3);
```

```text
        name         | len |         up          |         low         | first6 | last3 | mid | first_o
---------------------+-----+---------------------+---------------------+--------+-------+-----+---------
 Laptop Pro 14       |  13 | LAPTOP PRO 14       | laptop pro 14       | Laptop |  14   | Pro |       5
 Mechanical Keyboard |  19 | MECHANICAL KEYBOARD | mechanical keyboard | Mechan | ard   | cal |      16
(2 rows)
```

String positions in SQL start at **1**, not 0.

```sql
SELECT trim('   padded   ')                  AS trimmed,
       ltrim('xxhixx', 'x')                  AS ltrimmed,
       replace('2026-01-05', '-', '/')       AS replaced,
       lpad('42', 6, '0')                    AS zero_padded,
       initcap('ada LOVELACE')               AS initcapped,
       reverse('stressed')                   AS reversed,
       repeat('ab', 3)                       AS repeated;
```

```text
 trimmed | ltrimmed |  replaced  | zero_padded |  initcapped  | reversed | repeated
---------+----------+------------+-------------+--------------+----------+----------
 padded  | hixx     | 2026/01/05 | 000042      | Ada Lovelace | desserts | ababab
(1 row)
```

### Concatenation

`||` is the standard concatenation operator. It returns `NULL` if any piece is `NULL`. `concat()` treats `NULL` as an empty string, and `concat_ws()` ("with separator") joins pieces with a delimiter, skipping `NULL`s:

```sql
SELECT first_name || ' from ' || city            AS pipes,
       concat(first_name, ' from ', city)        AS concat_fn,
       concat_ws(', ', first_name, city, country) AS with_sep
FROM customers
WHERE customer_id IN (1, 6);
```

```text
       pipes       |     concat_fn     |     with_sep
-------------------+-------------------+-------------------
 Alice from London | Alice from London | Alice, London, UK
 NULL              | Farid from        | Farid, UK
(2 rows)
```

### Splitting and extracting

`split_part(text, delimiter, n)` returns the `n`th piece, which is ideal for emails, codes, and paths:

```sql
SELECT email,
       split_part(email, '@', 1) AS username,
       split_part(email, '@', 2) AS domain
FROM customers
LIMIT 3;
```

```text
       email       | username |   domain
-------------------+----------+-------------
 alice@example.com | alice    | example.com
 bruno@example.com | bruno    | example.com
 chloe@example.com | chloe    | example.com
(3 rows)
```

`format()` builds strings from a template, similar to Python's `%` formatting:

```sql
SELECT format('%s costs %s EUR (%s in stock)', name, price, stock) AS line
FROM products
WHERE product_id <= 2;
```

```text
                     line
-----------------------------------------------
 Laptop Pro 14 costs 1299.00 EUR (15 in stock)
 Laptop Air 13 costs 999.00 EUR (20 in stock)
(2 rows)
```

### Regular expressions

When `LIKE` is not expressive enough, PostgreSQL supports POSIX regular expressions. `~` matches case-sensitively, `~*` case-insensitively, and `!~` / `!~*` negate:

```sql
SELECT name FROM products WHERE name ~ '\d';                -- contains a digit
```

```text
     name
---------------
 Laptop Pro 14
 Laptop Air 13
 27in Monitor
(3 rows)
```

```sql
SELECT name FROM products WHERE name ~* '^(laptop|desktop)';  -- starts with either word, any case
```

```text
     name
---------------
 Laptop Pro 14
 Laptop Air 13
 Desktop Tower
(3 rows)
```

Regex functions extract and replace:

```sql
SELECT name,
       regexp_replace(name, '\s+', '_', 'g')          AS snake,
       substring(name FROM '\d+')                     AS first_number,
       regexp_count(name, '[aeiou]', 1, 'i')          AS vowels
FROM products
WHERE product_id IN (1, 6, 11);
```

```text
            name             |            snake            | first_number | vowels
-----------------------------+-----------------------------+--------------+--------
 Laptop Pro 14               | Laptop_Pro_14               | 14           |      3
 27in Monitor                | 27in_Monitor                | 27           |      4
 Noise-Cancelling Headphones | Noise-Cancelling_Headphones | NULL         |     10
(3 rows)
```

**Notes:**
- `||` with a `NULL` yields `NULL`, which silently blanks out whole labels. Use `concat`, `concat_ws`, or `COALESCE` ([Section 15](#15-coalesce-nullif-greatest-least)) when parts may be missing.
- `length` counts characters; `octet_length` counts bytes. They differ for non-ASCII text (`length('é')` is 1, `octet_length('é')` is 2 in UTF-8).
- Function names vary more across dialects for strings than for anything else: SQL Server uses `LEN`, `CHARINDEX`, and `+` for concatenation; MySQL uses `CONCAT` because `||` means `OR` there by default. The concepts are identical.
- Cleaning data at query time is fine for analysis. For application data, clean it once on the way in — and enforce the rule with a constraint — rather than on every read.
- `'g'` in `regexp_replace` means "global", replace every match. Without it, only the first match is replaced.

---

## 13. Numbers and Math

### Arithmetic operators

`+`, `-`, `*`, `/` behave as expected with one trap already mentioned: **integer divided by integer truncates**. `%` is the remainder (modulo) and `^` is exponentiation in PostgreSQL:

```sql
SELECT 17 / 5 AS int_div, 17 % 5 AS remainder, 17 / 5.0 AS real_div, 2 ^ 10 AS power, -7 / 2 AS neg_div;
```

```text
 int_div | remainder |      real_div      | power | neg_div
---------+-----------+--------------------+-------+---------
       3 |         2 | 3.4000000000000000 |  1024 |      -3
(1 row)
```

A common bug: computing a percentage from two integer counts gives 0.

```sql
SELECT 3 / 15 * 100            AS wrong_pct,
       3 * 100.0 / 15          AS right_pct,
       round(3 * 100.0 / 15, 1) AS rounded_pct;
```

```text
 wrong_pct |      right_pct      | rounded_pct
-----------+---------------------+-------------
         0 | 20.0000000000000000 |        20.0
(1 row)
```

### Rounding and friends

```sql
SELECT round(1234.5678, 2)   AS round_2,
       round(1234.5678, -2)  AS round_hundreds,
       trunc(1234.5678, 1)   AS truncated,
       ceil(4.1)             AS ceiling,
       floor(-4.1)           AS floor_neg,
       abs(-12)              AS absolute,
       sign(-12)             AS sign,
       sqrt(144)             AS root,
       ln(exp(1))            AS natural_log,
       log(1000)             AS log10;
```

```text
 round_2 | round_hundreds | truncated | ceiling | floor_neg | absolute | sign | root | natural_log | log10
---------+----------------+-----------+---------+-----------+----------+------+------+-------------+-------
 1234.57 |           1200 |    1234.5 |       5 |        -5 |       12 |   -1 |   12 |           1 |     3
(1 row)
```

### Division by zero

Dividing by zero is an error that aborts the whole query:

```sql
SELECT name, price / stock AS price_per_unit_in_stock FROM products;
```

```text
ERROR:  division by zero
```

The idiomatic fix is `NULLIF(denominator, 0)`, which turns a zero denominator into `NULL`, so the division yields `NULL` for that row instead of crashing the query:

```sql
SELECT name, round(price / NULLIF(stock, 0), 2) AS price_per_unit
FROM products
WHERE category_id = 4;
```

```text
            name             | price_per_unit
-----------------------------+----------------
 Mechanical Keyboard         |           1.80
 Wireless Mouse              |           0.25
 USB-C Hub                   |           NULL
 Noise-Cancelling Headphones |           7.96
 Webcam HD                   |           1.48
(5 rows)
```

### Bucketing numbers

`width_bucket` assigns values to equal-width bins, which is the SQL way to build a histogram, and integer division does the same for round intervals:

```sql
SELECT width_bucket(price, 0, 1500, 3) AS bucket,
       count(*)                        AS products,
       min(price), max(price)
FROM products
GROUP BY bucket
ORDER BY bucket;
```

```text
 bucket | products |   min   |   max
--------+----------+---------+---------
      1 |        9 |   29.99 |  249.00
      2 |        1 |  999.00 |  999.00
      3 |        2 | 1299.00 | 1499.00
(3 rows)
```

### Random numbers

`random()` returns a value in `[0, 1)`. It is useful for sampling and shuffling (`ORDER BY random()`), covered properly in [Section 50](#50-sql-for-machine-learning-and-ai-features-splits-and-vector-search). Since its output changes on every run, it is not shown here.

**Notes:**
- Make at least one operand decimal (`100.0`, or `::numeric`) whenever a ratio or percentage is intended.
- `round()` on `numeric` rounds half away from zero (`round(2.5)` is 3). On `double precision` it follows the platform's rounding, which may round half to even. Another reason to keep money in `numeric`.
- `NULLIF(x, 0)` in denominators is so common that it is worth typing by reflex.
- `^` is exponentiation in PostgreSQL but bitwise XOR in MySQL and SQL Server. `power(2, 10)` is portable.

---

## 14. Dates, Times, and Intervals

Dates are where dialects differ most and where subtle bugs live longest. The concepts, however, are universal — a type for calendar days, a type for moments in time, a type for durations, and functions to truncate, extract, format, and do arithmetic.

### The types

| Type | Example | Meaning |
|:--|:--|:--|
| `date` | `2026-03-15` | A calendar day, no time |
| `timestamp` | `2026-03-15 14:30:00` | A date and wall-clock time, **no time zone** |
| `timestamptz` | `2026-03-15 14:30:00+00` | An absolute **moment in time**, stored as UTC, displayed in the session's time zone |
| `interval` | `3 days 04:00:00` | A duration |
| `time` | `14:30:00` | A time of day, no date |

For anything that happened (orders, logins, events), use `timestamptz`. Despite the name, it does not store a time zone — it stores an unambiguous instant and converts on input and output. Plain `timestamp` is a wall-clock reading that means different instants in different places, which causes bugs the moment servers or users span time zones.

```sql
SELECT ordered_at,
       ordered_at AT TIME ZONE 'America/Toronto' AS toronto_wall_clock,
       ordered_at AT TIME ZONE 'Asia/Tokyo'      AS tokyo_wall_clock
FROM orders
WHERE order_id = 1;
```

```text
       ordered_at       | toronto_wall_clock  |  tokyo_wall_clock
------------------------+---------------------+---------------------
 2026-01-05 09:15:00+00 | 2026-01-05 04:15:00 | 2026-01-05 18:15:00
(1 row)
```

### Current date and time

`current_date`, `now()` (same as `current_timestamp`), and `localtimestamp` return the current date and time. `now()` is fixed at the **start of the transaction**, so every row in one statement sees the same value. Their output changes every run, so this guide uses fixed dates instead.

### Extracting parts

`extract(field FROM value)` (standard) and `date_part('field', value)` pull out a component:

```sql
SELECT order_id,
       ordered_at,
       extract(year FROM ordered_at)    AS yr,
       extract(month FROM ordered_at)   AS mon,
       extract(dow FROM ordered_at)     AS weekday,   -- 0 = Sunday
       extract(hour FROM ordered_at)    AS hr,
       extract(quarter FROM ordered_at) AS qtr
FROM orders
WHERE order_id IN (1, 5, 9);
```

```text
 order_id |       ordered_at       |  yr  | mon | weekday | hr | qtr
----------+------------------------+------+-----+---------+----+-----
        1 | 2026-01-05 09:15:00+00 | 2026 |   1 |       1 |  9 |   1
        5 | 2026-02-14 16:20:00+00 | 2026 |   2 |       6 | 16 |   1
        9 | 2026-03-30 21:05:00+00 | 2026 |   3 |       1 | 21 |   1
(3 rows)
```

### Truncating to a period

`date_trunc` rounds a timestamp **down** to the start of a unit. It is the workhorse of time-series reporting, because grouping by `date_trunc('month', ...)` gives one row per month:

```sql
SELECT date_trunc('month', ordered_at)::date AS month,
       count(*)                              AS orders
FROM orders
GROUP BY month
ORDER BY month;
```

```text
   month    | orders
------------+--------
 2026-01-01 |      3
 2026-02-01 |      3
 2026-03-01 |      3
 2026-04-01 |      2
 2026-05-01 |      2
 2026-06-01 |      2
(6 rows)
```

Units include `year`, `quarter`, `month`, `week` (weeks start on Monday), `day`, and `hour`.

### Date arithmetic

Subtracting two dates gives an integer number of days. Adding an integer to a date adds days. Anything involving timestamps uses `interval`:

```sql
SELECT '2026-03-15'::date - '2026-01-01'::date             AS days_between,
       '2026-01-31'::date + 7                              AS plus_week,
       '2026-01-31'::date + interval '1 month'             AS plus_month,
       '2026-03-15 10:00'::timestamp - interval '90 minutes' AS minus_90min,
       age('2026-06-20'::date, '2025-01-05'::date)          AS age_between;
```

```text
 days_between | plus_week  |     plus_month      |     minus_90min     |      age_between
--------------+------------+---------------------+---------------------+-----------------------
           73 | 2026-02-07 | 2026-02-28 00:00:00 | 2026-03-15 08:30:00 | 1 year 5 mons 15 days
(1 row)
```

Adding one month to January 31 gives February 28: PostgreSQL clamps to the last valid day. How long ago did each customer sign up, relative to a fixed reporting date?

```sql
SELECT first_name,
       signup_date,
       DATE '2026-07-01' - signup_date AS days_as_customer
FROM customers
ORDER BY days_as_customer DESC
LIMIT 3;
```

```text
 first_name | signup_date | days_as_customer
------------+-------------+------------------
 Alice      | 2025-01-05  |              542
 Bruno      | 2025-01-18  |              529
 Chloe      | 2025-02-02  |              514
(3 rows)
```

### Formatting and parsing

`to_char` formats dates and numbers as text; `to_date` and `to_timestamp` parse text into dates:

```sql
SELECT to_char(ordered_at, 'YYYY-MM-DD HH24:MI')   AS iso_minute,
       to_char(ordered_at, 'Dy DD Mon YYYY')       AS readable,
       to_char(ordered_at, 'YYYY-"Q"Q')            AS year_quarter,
       to_date('15/03/2026', 'DD/MM/YYYY')         AS parsed
FROM orders
WHERE order_id = 7;
```

```text
    iso_minute    |    readable     | year_quarter |   parsed
------------------+-----------------+--------------+------------
 2026-03-09 13:10 | Mon 09 Mar 2026 | 2026-Q1      | 2026-03-15
(1 row)
```

### Generating a calendar

`generate_series` creates a row per step. With dates, it builds a **date spine**: a complete list of periods that you then left-join your data onto, so that months with zero orders still appear as zero instead of vanishing:

```sql
SELECT gs::date AS month
FROM generate_series('2026-01-01'::date, '2026-06-01'::date, interval '1 month') AS gs;
```

```text
   month
------------
 2026-01-01
 2026-02-01
 2026-03-01
 2026-04-01
 2026-05-01
 2026-06-01
(6 rows)
```

[Section 31](#31-analytics-playbook-cohorts-retention-funnels-growth) uses this to build gap-free reports.

**Notes:**
- Store event times as `timestamptz`, store them in UTC-aware form, and convert to local time only for display.
- Filter time ranges with half-open intervals: `ts >= '2026-01-01' AND ts < '2026-02-01'`. It is correct for every precision and avoids the `BETWEEN` end-of-day trap.
- Applying a function to a column in `WHERE` (for example `WHERE date_trunc('month', ordered_at) = '2026-01-01'`) usually prevents an index on that column from being used. The half-open range form can use the index ([Section 42](#42-query-plans-and-performance)).
- Date functions are the least portable part of SQL. MySQL: `DATE_FORMAT`, `DATE_ADD`, `DATEDIFF`. SQL Server: `DATEADD`, `DATEDIFF`, `DATEPART`, `FORMAT`. SQLite: `date()`, `strftime()`. Look them up per dialect — the ideas are the same.
- `extract(dow ...)` numbers Sunday as 0. `extract(isodow ...)` numbers Monday as 1 through Sunday as 7, matching ISO weeks.

---

## 15. COALESCE, NULLIF, GREATEST, LEAST

These four small functions handle missing values and comparisons across columns. They are standard SQL (apart from minor dialect variants) and appear constantly in real queries.

### COALESCE: the first non-null value

`COALESCE(a, b, c, ...)` returns the first argument that is not `NULL`. Use it to supply defaults:

```sql
SELECT first_name,
       city,
       COALESCE(city, 'Unknown')              AS city_or_default,
       COALESCE(city, country)                AS best_location
FROM customers
WHERE customer_id IN (1, 6, 9);
```

```text
 first_name |  city  | city_or_default | best_location
------------+--------+-----------------+---------------
 Alice      | London | London          | London
 Farid      | NULL   | Unknown         | UK
 Isla       | NULL   | Unknown         | USA
(3 rows)
```

It is just as useful for values that go missing in an outer join, such as orders with no sales rep:

```sql
SELECT o.order_id,
       COALESCE(e.first_name, '(no rep)') AS sales_rep
FROM orders AS o
LEFT JOIN employees AS e ON e.employee_id = o.employee_id
WHERE o.order_id IN (5, 6, 13);
```

```text
 order_id | sales_rep
----------+-----------
        5 | Tom
       13 | (no rep)
        6 | (no rep)
(3 rows)
```

(That query uses a `LEFT JOIN`, explained in [Section 21](#21-outer-joins). The point here is the `COALESCE`.)

### NULLIF: turn a value into NULL

`NULLIF(a, b)` returns `NULL` if `a = b`, otherwise `a`. Its two main uses are safe division (seen in [Section 13](#13-numbers-and-math)) and treating sentinel values like empty strings as missing:

```sql
SELECT NULLIF('', '')          AS empty_to_null,
       NULLIF('Berlin', '')    AS kept,
       10 / NULLIF(0, 0)       AS safe_division;
```

```text
 empty_to_null |  kept  | safe_division
---------------+--------+---------------
 NULL          | Berlin |          NULL
(1 row)
```

`COALESCE(NULLIF(trim(x), ''), 'default')` is a common cleaning idiom — blank or whitespace-only text becomes a default.

### GREATEST and LEAST: row-wise max and min

`max` and `min` aggregate **down** a column across rows. `GREATEST` and `LEAST` compare **across** values within a single row:

```sql
SELECT name,
       price,
       GREATEST(price * 0.8, 40)   AS discounted_with_floor,
       LEAST(stock, 10)            AS display_stock_cap
FROM products
WHERE product_id IN (1, 4, 8);
```

```text
       name       |  price  | discounted_with_floor | display_stock_cap
------------------+---------+-----------------------+-------------------
 Laptop Pro 14    | 1299.00 |              1039.200 |                10
 Wireless Mouse   |   29.99 |                    40 |                10
 SQL for Everyone |   39.00 |                    40 |                10
(3 rows)
```

In PostgreSQL, `GREATEST` and `LEAST` ignore `NULL` arguments; in MySQL and Oracle, any `NULL` argument makes the result `NULL`. When portability matters, wrap arguments in `COALESCE`.

**Notes:**
- `COALESCE` is standard and portable. `IFNULL` (MySQL, SQLite), `ISNULL` (SQL Server), and `NVL` (Oracle) are two-argument dialect versions.
- `COALESCE` evaluates arguments lazily from left to right and stops at the first non-null one.
- All `COALESCE` arguments must have compatible types: `COALESCE(price, 'n/a')` fails because `'n/a'` is not numeric. Cast first: `COALESCE(price::text, 'n/a')`.
- Replacing `NULL` with 0 changes averages: `avg(COALESCE(x, 0))` counts missing values as zeros, while `avg(x)` ignores them. Decide which one the question actually needs.

---

## 16. Aggregate Functions

An **aggregate function** collapses many rows into one value — a count, a total, an average. Without `GROUP BY`, an aggregate treats the whole (filtered) table as one group and returns exactly one row.

### The core five

```sql
SELECT count(*)            AS products,
       sum(stock)          AS units_in_stock,
       avg(price)          AS avg_price,
       min(price)          AS cheapest,
       max(price)          AS priciest
FROM products;
```

```text
 products | units_in_stock |      avg_price       | cheapest | priciest
----------+----------------+----------------------+----------+----------
       12 |            735 | 384.3241666666666667 |    29.99 |  1499.00
(1 row)
```

`avg` on `numeric` returns many decimal places — wrap it in `round(avg(price), 2)` for display.

### The three kinds of count

`count` is subtle, and interviewers know it:

| Expression | Counts |
|:--|:--|
| `count(*)` | Rows, regardless of content |
| `count(col)` | Rows where `col` is **not NULL** |
| `count(DISTINCT col)` | Distinct non-null values of `col` |

```sql
SELECT count(*)                 AS all_customers,
       count(city)              AS with_city,
       count(DISTINCT city)     AS distinct_cities,
       count(DISTINCT country)  AS distinct_countries
FROM customers;
```

```text
 all_customers | with_city | distinct_cities | distinct_countries
---------------+-----------+-----------------+--------------------
            10 |         8 |               6 |                  6
(1 row)
```

### Aggregates ignore NULL

Every aggregate except `count(*)` skips `NULL` inputs. This is usually what you want, but it changes averages in ways worth knowing:

```sql
SELECT avg(x)               AS avg_ignoring_nulls,
       avg(COALESCE(x, 0))  AS avg_nulls_as_zero,
       sum(x)               AS total,
       count(x)             AS counted
FROM (VALUES (10), (20), (NULL)) AS t(x);
```

```text
 avg_ignoring_nulls  |  avg_nulls_as_zero  | total | counted
---------------------+---------------------+-------+---------
 15.0000000000000000 | 10.0000000000000000 |    30 |       2
(1 row)
```

(`VALUES` builds a small inline table, handy for experiments. `AS t(x)` names the table and its column.)

An aggregate over **zero rows** returns `NULL` for everything except `count`, which returns 0:

```sql
SELECT count(*) AS n, sum(price) AS total, max(price) AS top
FROM products
WHERE price > 10000;
```

```text
 n | total | top
---+-------+------
 0 |  NULL | NULL
(1 row)
```

### Aggregating expressions

The argument can be any expression. Revenue is quantity times the price actually paid:

```sql
SELECT sum(quantity * unit_price)            AS gross_revenue,
       sum(quantity)                         AS units_sold,
       round(avg(quantity * unit_price), 2)  AS avg_line_value
FROM order_items;
```

```text
 gross_revenue | units_sold | avg_line_value
---------------+------------+----------------
       8198.64 |         34 |         315.33
(1 row)
```

### Collecting values: string_agg and array_agg

These aggregates build one string or array from many rows. `ORDER BY` inside the call controls the order of the pieces:

```sql
SELECT string_agg(first_name, ', ' ORDER BY first_name) AS engineers,
       array_agg(salary ORDER BY salary DESC)            AS salaries
FROM employees
WHERE department = 'Engineering';
```

```text
       engineers        |                salaries
------------------------+-----------------------------------------
 Leo, Marcus, Mia, Omar | {150000.00,110000.00,95000.00,95000.00}
(1 row)
```

`bool_and` and `bool_or` answer "all?" and "any?" across a group:

```sql
SELECT bool_and(stock > 0) AS all_in_stock,
       bool_or(stock = 0)  AS any_out_of_stock
FROM products;
```

```text
 all_in_stock | any_out_of_stock
--------------+------------------
 f            | t
(1 row)
```

### Statistical aggregates

PostgreSQL includes the statistics that analysts and data scientists need, so summaries can be computed where the data lives instead of after exporting it:

```sql
SELECT round(stddev_samp(salary), 2)                              AS stddev,
       round(var_samp(salary), 0)                                 AS variance,
       percentile_cont(0.5) WITHIN GROUP (ORDER BY salary)         AS median,
       percentile_disc(0.9) WITHIN GROUP (ORDER BY salary)         AS p90,
       percentile_cont(ARRAY[0.25, 0.75]) WITHIN GROUP (ORDER BY salary) AS quartiles,
       mode() WITHIN GROUP (ORDER BY salary)                      AS most_common
FROM employees;
```

```text
  stddev  |  variance  | median |    p90    |   quartiles    | most_common
----------+------------+--------+-----------+----------------+-------------
 40884.32 | 1671527778 |  95000 | 180000.00 | {70000,120000} |    60000.00
(1 row)
```

`percentile_cont` interpolates between values (the statistical median); `percentile_disc` returns an actual value from the data. The `WITHIN GROUP (ORDER BY ...)` syntax marks these as *ordered-set aggregates*, which need sorted input.

Correlation and linear regression are aggregates too:

```sql
SELECT round(corr(price, stock)::numeric, 3)              AS corr_price_stock,
       round(regr_slope(stock, price)::numeric, 4)        AS slope,
       round(regr_intercept(stock, price)::numeric, 2)    AS intercept,
       round(regr_r2(stock, price)::numeric, 3)           AS r_squared
FROM products;
```

```text
 corr_price_stock |  slope  | intercept | r_squared
------------------+---------+-----------+-----------
           -0.505 | -0.0588 |     83.85 |     0.255
(1 row)
```

**Notes:**
- `count(*)` vs `count(col)` vs `count(DISTINCT col)` is a guaranteed interview question. Know all three and their `NULL` behavior.
- `sum` of no rows is `NULL`, not 0. Use `COALESCE(sum(x), 0)` when a report needs a zero.
- `avg` of an integer column returns `numeric` in PostgreSQL (no truncation), but in SQL Server `avg` of integers returns an integer. Cast defensively when porting.
- `stddev` and `variance` are aliases for the *sample* versions (`stddev_samp`, `var_samp`). Use `stddev_pop` for the population statistic.
- Median is `percentile_cont(0.5) WITHIN GROUP (ORDER BY x)` in PostgreSQL, Oracle, and SQL Server (where it is a window function). MySQL has no built-in median.

---

## 17. GROUP BY and HAVING

`GROUP BY` splits rows into groups that share the same values in the listed columns, then computes aggregates **per group**. The result has one row per group.

```sql
SELECT status, count(*) AS orders
FROM orders
GROUP BY status
ORDER BY orders DESC;
```

```text
  status   | orders
-----------+--------
 delivered |     11
 shipped   |      2
 cancelled |      1
 pending   |      1
(4 rows)
```

### Grouping by several columns

Each distinct *combination* forms a group:

```sql
SELECT country, city, count(*) AS customers
FROM customers
GROUP BY country, city
ORDER BY country, city;
```

```text
 country  |   city    | customers
----------+-----------+-----------
 Canada   | Toronto   |         2
 Germany  | Berlin    |         2
 Portugal | Lisbon    |         1
 Sweden   | Stockholm |         1
 UK       | London    |         1
 UK       | NULL      |         1
 USA      | Boston    |         1
 USA      | NULL      |         1
(8 rows)
```

Note that the two customers with no city formed their own groups within their countries: `GROUP BY` treats all `NULL`s as one value.

### The golden rule

Every column in the `SELECT` list must either be **inside an aggregate** or **listed in `GROUP BY`**. Otherwise the database would have to pick one value out of many, and it refuses:

```sql
SELECT department, first_name, max(salary)
FROM employees
GROUP BY department;
```

```text
ERROR:  column "employees.first_name" must appear in the GROUP BY clause or be used in an aggregate function
LINE 1: SELECT department, first_name, max(salary)
                           ^
```

Which `first_name` should represent the whole Engineering group? The question has no answer, hence the error. To find *who* earns the top salary in each department, you need a join, `DISTINCT ON`, or a window function ([Section 30](#30-classic-patterns-top-n-dedup-gaps-and-islands-pivots)).

PostgreSQL relaxes the rule in one sensible case — if you group by a table's primary key, you may select any other column of that table, because the key determines them all.

### Grouping by expressions

You can group by any expression, and PostgreSQL lets you refer to it by its output alias:

```sql
SELECT order_id,
       sum(quantity * unit_price) AS order_total,
       count(*)                   AS lines
FROM order_items
GROUP BY order_id
ORDER BY order_total DESC
LIMIT 5;
```

```text
 order_id | order_total | lines
----------+-------------+-------
       15 |     1498.00 |     2
       10 |     1388.90 |     2
        1 |     1228.99 |     2
        4 |     1103.98 |     3
       13 |      999.00 |     1
(5 rows)
```

```sql
SELECT extract(year FROM hire_date) AS hire_year, count(*) AS hires
FROM employees
GROUP BY hire_year
ORDER BY hire_year;
```

```text
 hire_year | hires
-----------+-------
      2019 |     2
      2020 |     2
      2021 |     2
      2022 |     1
      2023 |     1
      2024 |     1
(6 rows)
```

### HAVING: filtering groups

`WHERE` filters rows **before** grouping; `HAVING` filters groups **after** aggregation, so it can use aggregate values. Both can appear in the same query:

```sql
SELECT customer_id,
       count(*)        AS orders,
       min(ordered_at)::date AS first_order
FROM orders
WHERE status <> 'cancelled'          -- row filter: ignore cancelled orders
GROUP BY customer_id
HAVING count(*) >= 2                  -- group filter: repeat customers only
ORDER BY orders DESC, customer_id;
```

```text
 customer_id | orders | first_order
-------------+--------+-------------
           1 |      3 | 2026-01-05
           3 |      3 | 2026-02-03
           2 |      2 | 2026-01-12
           4 |      2 | 2026-02-14
(4 rows)
```

Put conditions that do not need aggregates in `WHERE`, not `HAVING`. The result is the same, but filtering early means less data to group.

**Notes:**
- The error "column must appear in the GROUP BY clause or be used in an aggregate function" always means the golden rule was broken. Decide whether the column should be a grouping key or aggregated.
- MySQL with the `ONLY_FULL_GROUP_BY` mode disabled (common in old installs) silently returns an arbitrary value for ungrouped columns. That is a bug factory; PostgreSQL's refusal is correct.
- `GROUP BY` does not guarantee order. Add `ORDER BY` if order matters.
- `COUNT(DISTINCT ...)` inside a grouped query answers questions like "how many distinct products did each customer buy?"
- Grouping by position (`GROUP BY 1, 2`) is convenient in ad-hoc analysis and fragile in saved code, because reordering the select list changes the grouping.

---

## 18. FILTER, ROLLUP, CUBE, and GROUPING SETS

These extensions turn one grouped query into a small report — several filtered measures side by side, plus subtotals and grand totals.

### FILTER: conditional aggregates, cleanly

`FILTER (WHERE ...)` restricts which rows feed a single aggregate. It is the readable, standard replacement for `sum(CASE WHEN ...)` from [Section 10](#10-expressions-and-case):

```sql
SELECT date_trunc('month', ordered_at)::date                AS month,
       count(*)                                           AS all_orders,
       count(*) FILTER (WHERE status = 'delivered')       AS delivered,
       count(*) FILTER (WHERE status = 'cancelled')       AS cancelled,
       count(*) FILTER (WHERE employee_id IS NULL)        AS self_service
FROM orders
GROUP BY month
ORDER BY month;
```

```text
   month    | all_orders | delivered | cancelled | self_service
------------+------------+-----------+-----------+--------------
 2026-01-01 |          3 |         3 |         0 |            0
 2026-02-01 |          3 |         2 |         1 |            1
 2026-03-01 |          3 |         3 |         0 |            0
 2026-04-01 |          2 |         2 |         0 |            0
 2026-05-01 |          2 |         1 |         0 |            1
 2026-06-01 |          2 |         0 |         0 |            0
(6 rows)
```

`FILTER` is standard SQL supported by PostgreSQL, SQLite, and DuckDB. In MySQL and SQL Server, use the `CASE` form.

### ROLLUP: subtotals and a grand total

`GROUP BY ROLLUP (a, b)` produces groups for `(a, b)`, then subtotals for `(a)`, then a grand total `()`. The rolled-up columns show `NULL` in subtotal rows:

```sql
SELECT department,
       title,
       count(*)     AS people,
       sum(salary)  AS payroll
FROM employees
GROUP BY ROLLUP (department, title)
ORDER BY department, title;
```

```text
 department  |      title      | people |  payroll
-------------+-----------------+--------+-----------
 Engineering | CTO             |      1 | 150000.00
 Engineering | Engineer        |      2 | 190000.00
 Engineering | Senior Engineer |      1 | 110000.00
 Engineering | NULL            |      4 | 450000.00
 Executive   | CEO             |      1 | 180000.00
 Executive   | NULL            |      1 | 180000.00
 Sales       | Head of Sales   |      1 | 120000.00
 Sales       | Sales Rep       |      2 | 120000.00
 Sales       | NULL            |      3 | 240000.00
 Support     | Support Lead    |      1 |  70000.00
 Support     | NULL            |      1 |  70000.00
 NULL        | NULL            |      9 | 940000.00
(12 rows)
```

The `NULL` in `title` marks each department subtotal, and the row with both `NULL` is the grand total.

### GROUPING(): telling subtotal NULLs from real NULLs

If a grouping column can itself contain `NULL`, you cannot tell a subtotal row from a real `NULL` group. `GROUPING(col)` returns 1 when the column was rolled up in that row, 0 otherwise, which lets you label rows properly:

```sql
SELECT CASE WHEN GROUPING(department) = 1 THEN 'ALL DEPARTMENTS' ELSE department END AS department,
       count(*)    AS people,
       sum(salary) AS payroll
FROM employees
GROUP BY ROLLUP (department)
ORDER BY GROUPING(department), department;
```

```text
   department    | people |  payroll
-----------------+--------+-----------
 Engineering     |      4 | 450000.00
 Executive       |      1 | 180000.00
 Sales           |      3 | 240000.00
 Support         |      1 |  70000.00
 ALL DEPARTMENTS |      9 | 940000.00
(5 rows)
```

### CUBE: every combination

`CUBE (a, b)` produces subtotals for every combination: `(a, b)`, `(a)`, `(b)`, and `()`. Useful for cross-tabulated summaries:

```sql
SELECT status,
       (employee_id IS NOT NULL) AS has_rep,
       count(*)                  AS orders
FROM orders
GROUP BY CUBE (status, has_rep)
ORDER BY status NULLS LAST, has_rep NULLS LAST;
```

```text
  status   | has_rep | orders
-----------+---------+--------
 cancelled | f       |      1
 cancelled | NULL    |      1
 delivered | t       |     11
 delivered | NULL    |     11
 pending   | t       |      1
 pending   | NULL    |      1
 shipped   | f       |      1
 shipped   | t       |      1
 shipped   | NULL    |      2
 NULL      | f       |      2
 NULL      | t       |     13
 NULL      | NULL    |     15
(12 rows)
```

### GROUPING SETS: exactly the groupings you list

`ROLLUP` and `CUBE` are shorthands for `GROUPING SETS`, which lets you name each grouping explicitly. This query computes per-country and per-city counts plus a total in one pass:

```sql
SELECT country, city, count(*) AS customers
FROM customers
GROUP BY GROUPING SETS ((country), (city), ())
ORDER BY country NULLS LAST, city NULLS LAST;
```

```text
 country  |   city    | customers
----------+-----------+-----------
 Canada   | NULL      |         2
 Germany  | NULL      |         2
 Portugal | NULL      |         1
 Sweden   | NULL      |         1
 UK       | NULL      |         2
 USA      | NULL      |         2
 NULL     | Berlin    |         2
 NULL     | Boston    |         1
 NULL     | Lisbon    |         1
 NULL     | London    |         1
 NULL     | Stockholm |         1
 NULL     | Toronto   |         2
 NULL     | NULL      |         2
 NULL     | NULL      |        10
(14 rows)
```

Look at the last two rows. Both show `NULL, NULL`, but one is the group of customers whose city is unknown (2) and the other is the grand total (10). This is exactly the ambiguity that `GROUPING()` resolves.

**Notes:**
- `FILTER` beats `CASE` inside aggregates for readability — use it wherever it is supported.
- `ROLLUP (a, b)` is hierarchical (a, then a+b); `CUBE (a, b)` is every combination. `ROLLUP` of `n` columns gives `n + 1` grouping levels; `CUBE` gives `2^n`.
- Always use `GROUPING()` in production reports so that genuine `NULL` groups are not mistaken for totals.
- SQL Server supports all three; MySQL supports only `WITH ROLLUP` (`GROUP BY a, b WITH ROLLUP`). BI tools often generate these clauses behind the scenes.

---

## 19. Keys and Relationships

Real data is split across many tables, and **keys** are what connect them. Understanding keys is understanding why joins work.

### Primary keys

A **primary key** (PK) uniquely identifies each row in a table. It must be unique and not `NULL`, and a table has at most one. In the sample database, `customers.customer_id` is a primary key — no two customers share one, and every customer has one.

There are two styles:

- A **surrogate key** is a meaningless generated number or UUID (`customer_id`). It never changes, is small, and makes joins cheap. This is the default choice.
- A **natural key** is real-world data that happens to be unique — an email, an ISBN, a country code. It carries meaning but can change (people change emails) and is often wide.

The common pattern is a surrogate primary key *plus* a `UNIQUE` constraint on the natural key, as the sample database does with `customers.email`.

A **composite key** uses several columns together. `order_items` has the primary key `(order_id, product_id)` — an order can contain a product at most once, and the pair identifies the line.

### Foreign keys

A **foreign key** (FK) is a column whose values must match a primary key (or unique key) in another table. `orders.customer_id REFERENCES customers (customer_id)` guarantees that every order belongs to a customer who exists. The database enforces this on every write:

```sql
INSERT INTO orders (customer_id, ordered_at, status)
VALUES (999, '2026-07-01', 'pending');
```

```text
ERROR:  insert or update on table "orders" violates foreign key constraint "orders_customer_id_fkey"
DETAIL:  Key (customer_id)=(999) is not present in table "customers".
```

### Relationship types

| Relationship | Meaning | How it is modeled | Example |
|:--|:--|:--|:--|
| One-to-many (1:N) | One row relates to many rows elsewhere | A foreign key on the "many" side | One customer, many orders (`orders.customer_id`) |
| Many-to-many (M:N) | Many relate to many | A **junction table** holding two foreign keys | Orders and products, via `order_items` |
| One-to-one (1:1) | Exactly one on each side | A foreign key that is also unique (or a shared primary key) | A user and their profile settings |
| Self-referencing | A row relates to another row in the same table | A foreign key pointing to its own table | `employees.manager_id`, `categories.parent_id` |

The many-to-many case deserves attention because it is everywhere — students and courses, users and roles, orders and products. A relational table cannot store a list in one cell (well, not cleanly), so the relationship itself becomes a table. The junction table often carries its own data too — `order_items` stores the quantity and the price paid.

**Notes:**
- A primary key implies `UNIQUE` and `NOT NULL`, and PostgreSQL automatically creates an index for it. Foreign keys do **not** get an automatic index, which matters for join and delete performance ([Section 39](#39-indexes)).
- Foreign keys can be `NULL` unless declared `NOT NULL`. `orders.employee_id` is nullable because some orders have no sales rep.
- "Surrogate vs natural key" is a common design interview question. Default to surrogate keys with unique constraints on natural identifiers.
- Some analytics warehouses accept primary and foreign key declarations but do not enforce them (they are hints for the optimizer). Never assume integrity in a warehouse — test for it ([Section 48](#48-data-engineering-warehouses-star-schemas-and-pipelines)).

---

## 20. INNER JOIN

A **join** combines rows from two tables into wider rows, pairing them according to a condition. An **inner join** keeps only the pairs where the condition is true; rows with no partner on the other side disappear.

```sql
SELECT o.order_id, o.status, c.first_name, c.country
FROM orders AS o
INNER JOIN customers AS c ON c.customer_id = o.customer_id
WHERE o.order_id <= 5;
```

```text
 order_id |  status   | first_name | country
----------+-----------+------------+----------
        1 | delivered | Alice      | UK
        2 | delivered | Bruno      | Portugal
        3 | delivered | Alice      | UK
        4 | delivered | Chloe      | Canada
        5 | delivered | Daniel     | Germany
(5 rows)
```

Conceptually, the database considers every pair of rows (every order with every customer), keeps the pairs where `c.customer_id = o.customer_id`, and then applies `WHERE`. Physically it is far smarter — hash joins, merge joins, index lookups — but that mental model always gives the right answer. `INNER` is optional: `JOIN` alone means inner join.

### Joining many tables

Each additional `JOIN` adds another table to the row. To see what each order contained, walk the relationships: orders to order items to products:

```sql
SELECT o.order_id,
       c.first_name,
       p.name       AS product,
       oi.quantity,
       oi.unit_price,
       oi.quantity * oi.unit_price AS line_total
FROM orders AS o
JOIN customers   AS c  ON c.customer_id = o.customer_id
JOIN order_items AS oi ON oi.order_id   = o.order_id
JOIN products    AS p  ON p.product_id  = oi.product_id
WHERE o.order_id IN (4, 9)
ORDER BY o.order_id, p.name;
```

```text
 order_id | first_name |        product         | quantity | unit_price | line_total
----------+------------+------------------------+----------+------------+------------
        4 | Chloe      | Laptop Air 13          |        1 |     999.00 |     999.00
        4 | Chloe      | USB-C Hub              |        1 |      45.00 |      45.00
        4 | Chloe      | Wireless Mouse         |        2 |      29.99 |      59.98
        9 | Alice      | Designing Data Systems |        1 |      55.00 |      55.00
        9 | Alice      | Python Deep Dive       |        1 |      49.00 |      49.00
        9 | Alice      | SQL for Everyone       |        1 |      39.00 |      39.00
(6 rows)
```

Joins combine naturally with grouping. Revenue per category, excluding cancelled orders:

```sql
SELECT cat.name                          AS category,
       count(DISTINCT o.order_id)        AS orders,
       sum(oi.quantity)                  AS units,
       sum(oi.quantity * oi.unit_price)  AS revenue
FROM order_items AS oi
JOIN orders      AS o   ON o.order_id      = oi.order_id
JOIN products    AS p   ON p.product_id    = oi.product_id
JOIN categories  AS cat ON cat.category_id = p.category_id
WHERE o.status <> 'cancelled'
GROUP BY cat.name
ORDER BY revenue DESC;
```

```text
  category   | orders | units | revenue
-------------+--------+-------+---------
 Laptops     |      5 |     5 | 5795.00
 Accessories |      9 |    13 |  937.64
 Computers   |      2 |     3 |  747.00
 Programming |      4 |    12 |  520.00
(4 rows)
```

### USING

When the join columns have the same name in both tables, `USING (col)` is a shorter form, and the shared column appears only once in `SELECT *`:

```sql
SELECT order_id, first_name, status
FROM orders
JOIN customers USING (customer_id)
WHERE order_id = 1;
```

```text
 order_id | first_name |  status
----------+------------+-----------
        1 | Alice      | delivered
(1 row)
```

### The fan-out trap

A join to a "many" side multiplies rows. If you then aggregate a value from the "one" side, you count it once per matching row. This is the most common way to get silently wrong totals in SQL:

```sql
-- How many orders did Alice place? Joining to order_items first inflates the count.
SELECT c.first_name,
       count(*)                    AS wrong_order_count,
       count(DISTINCT o.order_id)  AS right_order_count
FROM customers   AS c
JOIN orders      AS o  ON o.customer_id = c.customer_id
JOIN order_items AS oi ON oi.order_id   = o.order_id
WHERE c.customer_id = 1
GROUP BY c.first_name;
```

```text
 first_name | wrong_order_count | right_order_count
------------+-------------------+-------------------
 Alice      |                 6 |                 3
(1 row)
```

Alice has 3 orders with 6 lines between them. The join produced 6 rows, so `count(*)` said 6. The safe habits: know the grain (what one row represents) of every table you join, aggregate at the right grain before joining when needed ([Section 25](#25-common-table-expressions)), and treat an unexpected `DISTINCT` as a warning sign.

### Old-style comma joins

You will see this legacy syntax in older code:

```sql
SELECT o.order_id, c.first_name
FROM orders o, customers c
WHERE c.customer_id = o.customer_id AND o.order_id = 2;
```

```text
 order_id | first_name
----------+------------
        2 | Bruno
(1 row)
```

It works, but mixing join conditions into `WHERE` makes it easy to forget one and produce an accidental cross join of every row with every row. Use explicit `JOIN ... ON`.

**Notes:**
- Always qualify columns with table aliases in multi-table queries (`o.status`, not `status`). If two tables share a column name, an unqualified reference is an "ambiguous column" error; if they do not yet, they might after the next migration.
- An inner join drops unmatched rows from **both** sides. If the question includes "even those without...", you need an outer join ([Section 21](#21-outer-joins)).
- Before trusting a total from a joined query, ask "what does one row of this result represent?" If the answer is "an order line", then counting rows counts lines, not orders.
- Joins on columns of different types (an `integer` to a `text` ID) either fail or cannot use indexes. Keep key types identical across tables.

---

## 21. Outer Joins

An **outer join** keeps rows even when they have no match, filling the missing side's columns with `NULL`.

| Join | Keeps |
|:--|:--|
| `INNER JOIN` | Only matched pairs |
| `LEFT JOIN` | Every row from the left table, matched or not |
| `RIGHT JOIN` | Every row from the right table, matched or not |
| `FULL JOIN` | Every row from both tables |

```text
 ◀─────────────── customers (left) ────────────────▶
                             ◀──────────────── orders (right) ─────────────────▶
┌───────────────────────────┬───────────────────────┬───────────────────────────┐
│ customers with no orders  │ matched customer and  │ orders with no customer   │
│ (Isla, Jonas)             │ order pairs           │ (none: the FK forbids it) │
└───────────────────────────┴───────────────────────┴───────────────────────────┘
  INNER JOIN = middle only        LEFT JOIN  = left + middle
  RIGHT JOIN = middle + right     FULL JOIN  = all three
```

### LEFT JOIN

Every customer, with their order count, including customers who never ordered:

```sql
SELECT c.customer_id,
       c.first_name,
       count(o.order_id) AS orders
FROM customers AS c
LEFT JOIN orders AS o ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.first_name
ORDER BY orders DESC, c.customer_id;
```

```text
 customer_id | first_name | orders
-------------+------------+--------
           1 | Alice      |      3
           3 | Chloe      |      3
           2 | Bruno      |      2
           4 | Daniel     |      2
           5 | Emma       |      2
           6 | Farid      |      1
           7 | Grace      |      1
           8 | Hugo       |      1
           9 | Isla       |      0
          10 | Jonas      |      0
(10 rows)
```

Notice `count(o.order_id)`, not `count(*)`. For Isla and Jonas, the join produced one row with `NULL` order columns; `count(*)` would count that row as 1, while `count(o.order_id)` correctly counts the `NULL` as nothing.

Here is what the unmatched rows look like before grouping:

```sql
SELECT c.first_name, o.order_id, o.status
FROM customers AS c
LEFT JOIN orders AS o ON o.customer_id = c.customer_id
WHERE c.customer_id IN (8, 9, 10);
```

```text
 first_name | order_id | status
------------+----------+---------
 Hugo       |       14 | shipped
 Jonas      |     NULL | NULL
 Isla       |     NULL | NULL
(3 rows)
```

### The ON vs WHERE trap

With inner joins, a condition behaves the same in `ON` or `WHERE`. With outer joins, it does not. **A `WHERE` condition on the right table's columns turns a left join back into an inner join**, because it filters out the `NULL`-extended rows:

```sql
-- Goal: every customer with their count of DELIVERED orders.
-- Wrong: the WHERE removes customers with no delivered orders (their status is NULL).
SELECT c.first_name, count(o.order_id) AS delivered
FROM customers AS c
LEFT JOIN orders AS o ON o.customer_id = c.customer_id
WHERE o.status = 'delivered'
GROUP BY c.customer_id, c.first_name
ORDER BY c.customer_id;
```

```text
 first_name | delivered
------------+-----------
 Alice      |         3
 Bruno      |         2
 Chloe      |         2
 Daniel     |         1
 Emma       |         1
 Farid      |         1
 Grace      |         1
(7 rows)
```

```sql
-- Right: put the condition in ON, so it decides what matches, not which rows survive.
SELECT c.first_name, count(o.order_id) AS delivered
FROM customers AS c
LEFT JOIN orders AS o ON o.customer_id = c.customer_id AND o.status = 'delivered'
GROUP BY c.customer_id, c.first_name
ORDER BY c.customer_id;
```

```text
 first_name | delivered
------------+-----------
 Alice      |         3
 Bruno      |         2
 Chloe      |         2
 Daniel     |         1
 Emma       |         1
 Farid      |         1
 Grace      |         1
 Hugo       |         0
 Isla       |         0
 Jonas      |         0
(10 rows)
```

The first query lost Hugo, Isla, and Jonas. The rule: conditions on the **optional** side of an outer join belong in `ON`; conditions on the **preserved** side belong in `WHERE`.

### RIGHT JOIN

`A RIGHT JOIN B` is identical to `B LEFT JOIN A`. Most people always write `LEFT JOIN` and order the tables accordingly, which keeps queries readable from top to bottom. Every category with its product count, including those with no products assigned directly (parent categories like `Electronics` hold none themselves, and `Home` is empty):

```sql
SELECT cat.name, count(p.product_id) AS products
FROM products AS p
RIGHT JOIN categories AS cat ON cat.category_id = p.category_id
GROUP BY cat.name
ORDER BY products, cat.name;
```

```text
    name     | products
-------------+----------
 Books       |        0
 Electronics |        0
 Home        |        0
 Computers   |        2
 Laptops     |        2
 Programming |        3
 Accessories |        5
(7 rows)
```

### FULL OUTER JOIN

A full join keeps unmatched rows from both sides. It is the tool for **reconciliation**: comparing two sources and finding what is missing from each. Which products have never been ordered, and are there order lines pointing at missing products?

```sql
SELECT p.product_id, p.name, oi.order_id
FROM products AS p
FULL JOIN order_items AS oi ON oi.product_id = p.product_id
WHERE p.product_id IS NULL OR oi.order_id IS NULL;
```

```text
 product_id |     name      | order_id
------------+---------------+----------
         12 | Webcam HD     |     NULL
          7 | Desktop Tower |     NULL
(2 rows)
```

Only the left-side gaps appear here, because foreign keys guarantee no order line points at a missing product. Across two systems without such guarantees, both kinds of gap appear.

**Notes:**
- In a `LEFT JOIN`, count a column from the right table (`count(o.order_id)`), never `count(*)`, when you want matches.
- "Conditions on the optional side go in `ON`" is the rule that prevents the most outer-join bugs. Interviewers love asking why a left join "stopped working".
- MySQL has no `FULL JOIN`. Emulate it with `LEFT JOIN ... UNION ... RIGHT JOIN` (using `UNION`, not `UNION ALL`, to drop the duplicated matched rows).
- A chain like `A LEFT JOIN B ... JOIN C ON c.x = b.x` quietly becomes inner, because the inner join to `C` drops rows where `B` was `NULL`. Once a chain goes outer, later joins that depend on it usually must be outer too.

---

## 22. Self, Cross, Semi, Anti, and Lateral Joins

These are not new syntax so much as new *uses* of joins, and each one answers a classic kind of question.

### Self join: a table joined to itself

A self-referencing table like `employees` needs to be joined to itself to put an employee next to their manager. Two aliases make the two roles clear:

```sql
SELECT e.first_name  AS employee,
       e.title,
       m.first_name  AS manager
FROM employees AS e
LEFT JOIN employees AS m ON m.employee_id = e.manager_id
ORDER BY e.employee_id;
```

```text
 employee |      title      | manager
----------+-----------------+---------
 Sofia    | CEO             | NULL
 Marcus   | CTO             | Sofia
 Priya    | Head of Sales   | Sofia
 Leo      | Senior Engineer | Marcus
 Mia      | Engineer        | Marcus
 Omar     | Engineer        | Leo
 Nina     | Sales Rep       | Priya
 Tom      | Sales Rep       | Priya
 Eva      | Support Lead    | Sofia
(9 rows)
```

The `LEFT JOIN` keeps Sofia, who has no manager. A classic interview question built on this: employees who earn more than their manager (none here, which is also a valid answer):

```sql
SELECT e.first_name, e.salary, m.first_name AS manager, m.salary AS manager_salary
FROM employees AS e
JOIN employees AS m ON m.employee_id = e.manager_id
WHERE e.salary > m.salary;
```

```text
 first_name | salary | manager | manager_salary
------------+--------+---------+----------------
(0 rows)
```

Self joins also compare rows within one table, such as pairs of employees in the same department (the `<` avoids pairing someone with themselves and listing each pair twice):

```sql
SELECT a.first_name, b.first_name, a.department
FROM employees AS a
JOIN employees AS b ON a.department = b.department AND a.employee_id < b.employee_id
WHERE a.department = 'Sales';
```

```text
 first_name | first_name | department
------------+------------+------------
 Priya      | Nina       | Sales
 Priya      | Tom        | Sales
 Nina       | Tom        | Sales
(3 rows)
```

### Cross join: every combination

`CROSS JOIN` pairs every row of one table with every row of the other (a Cartesian product), so 3 rows by 4 rows gives 12. It is useful for generating grids, such as every department for every quarter, which a report then fills in:

```sql
SELECT d.department, q.quarter
FROM (SELECT DISTINCT department FROM employees) AS d
CROSS JOIN (VALUES ('Q1'), ('Q2')) AS q(quarter)
ORDER BY d.department, q.quarter;
```

```text
 department  | quarter
-------------+---------
 Engineering | Q1
 Engineering | Q2
 Executive   | Q1
 Executive   | Q2
 Sales       | Q1
 Sales       | Q2
 Support     | Q1
 Support     | Q2
(8 rows)
```

An *accidental* cross join (a forgotten join condition) is a classic performance disaster — two tables of a million rows produce a trillion.

### Semi join: rows that have a match (EXISTS)

A semi join returns rows from one table that have **at least one** match in another, without duplicating them. `EXISTS` expresses it directly. Customers who have ordered at least once:

```sql
SELECT c.customer_id, c.first_name
FROM customers AS c
WHERE EXISTS (
    SELECT 1 FROM orders AS o WHERE o.customer_id = c.customer_id
)
ORDER BY c.customer_id;
```

```text
 customer_id | first_name
-------------+------------
           1 | Alice
           2 | Bruno
           3 | Chloe
           4 | Daniel
           5 | Emma
           6 | Farid
           7 | Grace
           8 | Hugo
(8 rows)
```

An inner join would return Alice three times (once per order) and need a `DISTINCT`. `EXISTS` stops at the first match and returns each customer once. The `SELECT 1` inside is a convention — `EXISTS` only checks whether any row comes back.

### Anti join: rows with no match (NOT EXISTS)

An anti join returns rows that have **no** match. Products that have never been ordered, written three ways:

```sql
-- 1. NOT EXISTS: clear, NULL-safe, usually the best plan
SELECT p.product_id, p.name
FROM products AS p
WHERE NOT EXISTS (SELECT 1 FROM order_items AS oi WHERE oi.product_id = p.product_id);
```

```text
 product_id |     name
------------+---------------
         12 | Webcam HD
          7 | Desktop Tower
(2 rows)
```

```sql
-- 2. LEFT JOIN ... IS NULL: equivalent and common
SELECT p.product_id, p.name
FROM products AS p
LEFT JOIN order_items AS oi ON oi.product_id = p.product_id
WHERE oi.product_id IS NULL;
```

```text
 product_id |     name
------------+---------------
         12 | Webcam HD
          7 | Desktop Tower
(2 rows)
```

```sql
-- 3. NOT IN: works here, but breaks if the subquery can return NULL
SELECT product_id, name
FROM products
WHERE product_id NOT IN (SELECT product_id FROM order_items);
```

```text
 product_id |     name
------------+---------------
          7 | Desktop Tower
         12 | Webcam HD
(2 rows)
```

Now the trap from [Section 8](#8-null-and-three-valued-logic), for real. Which employees are not anyone's manager? `manager_id` contains a `NULL` (Sofia's), and `NOT IN` returns nothing:

```sql
SELECT first_name FROM employees
WHERE employee_id NOT IN (SELECT manager_id FROM employees);
```

```text
 first_name
------------
(0 rows)
```

```sql
SELECT e.first_name FROM employees AS e
WHERE NOT EXISTS (SELECT 1 FROM employees AS r WHERE r.manager_id = e.employee_id);
```

```text
 first_name
------------
 Mia
 Tom
 Omar
 Eva
 Nina
(5 rows)
```

Use `NOT EXISTS` for anti joins, always.

### LATERAL: a subquery per row

A `LATERAL` subquery can refer to columns of tables that appear earlier in the `FROM` clause, so it runs once per outer row. It answers "top N per group" and "latest X per Y" elegantly. Each customer's most recent order:

```sql
SELECT c.first_name, last_order.order_id, last_order.ordered_at::date
FROM customers AS c
CROSS JOIN LATERAL (
    SELECT o.order_id, o.ordered_at
    FROM orders AS o
    WHERE o.customer_id = c.customer_id
    ORDER BY o.ordered_at DESC
    LIMIT 1
) AS last_order
ORDER BY c.customer_id;
```

```text
 first_name | order_id | ordered_at
------------+----------+------------
 Alice      |        9 | 2026-03-30
 Bruno      |       11 | 2026-04-25
 Chloe      |       15 | 2026-06-20
 Daniel     |       13 | 2026-05-19
 Emma       |       12 | 2026-05-06
 Farid      |        8 | 2026-03-15
 Grace      |       10 | 2026-04-11
 Hugo       |       14 | 2026-06-02
(8 rows)
```

`CROSS JOIN LATERAL` drops customers with no orders; `LEFT JOIN LATERAL (...) ON true` keeps them. SQL Server spells this `CROSS APPLY` and `OUTER APPLY`.

### Non-equi joins

Join conditions need not be equality. Joining to a table of ranges assigns each row to a band:

```sql
SELECT p.name, p.price, b.band
FROM products AS p
JOIN (VALUES ('budget', 0, 100), ('mid', 100, 1000), ('premium', 1000, 100000))
     AS b(band, low, high)
  ON p.price >= b.low AND p.price < b.high
WHERE p.category_id IN (3, 4)
ORDER BY p.price;
```

```text
            name             |  price  |  band
-----------------------------+---------+---------
 Wireless Mouse              |   29.99 | budget
 USB-C Hub                   |   45.00 | budget
 Webcam HD                   |   59.00 | budget
 Mechanical Keyboard         |   89.90 | budget
 Noise-Cancelling Headphones |  199.00 | mid
 Laptop Air 13               |  999.00 | mid
 Laptop Pro 14               | 1299.00 | premium
(7 rows)
```

**Notes:**
- Self joins need two different aliases for the same table — that is all they are.
- Prefer `EXISTS` / `NOT EXISTS` for "has any" and "has none" questions. They never duplicate rows and handle `NULL` correctly.
- `NOT IN (subquery)` is a latent bug whenever the subquery's column is nullable, even if it has no `NULL`s today.
- `LATERAL` is supported in PostgreSQL, MySQL 8, Oracle, DuckDB, and Snowflake; SQL Server uses `APPLY`.
- Cross joins are legitimate for building grids and spines. When a query is unexpectedly huge and slow, check for a missing join condition first.

---

## 23. UNION, INTERSECT, EXCEPT

Joins combine tables **side by side** (more columns). Set operations combine query results **top to bottom** (more rows), treating each result as a set of rows.

| Operator | Returns |
|:--|:--|
| `UNION` | Rows in either result, **duplicates removed** |
| `UNION ALL` | Rows in either result, **duplicates kept** |
| `INTERSECT` | Rows in both results |
| `EXCEPT` | Rows in the first result but not the second (`MINUS` in Oracle) |

The rules: both queries must return the same number of columns with compatible types, column names come from the first query, and a single `ORDER BY` at the end sorts the combined result.

```sql
-- One contact list from two tables
SELECT first_name, last_name, 'customer' AS kind FROM customers WHERE country = 'Germany'
UNION ALL
SELECT first_name, last_name, 'employee'         FROM employees WHERE department = 'Sales'
ORDER BY kind, first_name;
```

```text
 first_name | last_name |   kind
------------+-----------+----------
 Daniel     | Okafor    | customer
 Jonas      | Weber     | customer
 Nina       | Park      | employee
 Priya      | Nair      | employee
 Tom        | Walsh     | employee
(5 rows)
```

### UNION vs UNION ALL

`UNION` must sort or hash the whole result to remove duplicates. If you know the parts cannot overlap, or you want duplicates, `UNION ALL` is faster and more honest:

```sql
SELECT country FROM customers WHERE city = 'Toronto'
UNION
SELECT country FROM customers WHERE city IS NULL;
```

```text
 country
---------
 USA
 UK
 Canada
(3 rows)
```

```sql
SELECT country FROM customers WHERE city = 'Toronto'
UNION ALL
SELECT country FROM customers WHERE city IS NULL;
```

```text
 country
---------
 Canada
 Canada
 UK
 USA
(4 rows)
```

### INTERSECT and EXCEPT

Products bought both in January and in March:

```sql
SELECT oi.product_id FROM order_items oi JOIN orders o USING (order_id)
WHERE o.ordered_at >= '2026-01-01' AND o.ordered_at < '2026-02-01'
INTERSECT
SELECT oi.product_id FROM order_items oi JOIN orders o USING (order_id)
WHERE o.ordered_at >= '2026-03-01' AND o.ordered_at < '2026-04-01';
```

```text
 product_id
------------
          8
          9
(2 rows)
```

Countries where the shop has customers but no delivered order yet (a typical "in A, not in B" question):

```sql
SELECT country FROM customers
EXCEPT
SELECT c.country FROM customers c JOIN orders o USING (customer_id) WHERE o.status = 'delivered'
ORDER BY country;
```

```text
 country
---------
 Sweden
(1 row)
```

Set operations compare whole rows and treat `NULL`s as equal to each other (unlike `=`), which makes `EXCEPT` a convenient way to diff two tables with identical structure: `(SELECT * FROM a EXCEPT SELECT * FROM b)` lists rows in `a` missing from `b`.

**Notes:**
- Default to `UNION ALL`. Use `UNION` only when you actually need deduplication, and know that it costs a sort or hash.
- Columns are matched by **position**, not by name. Swapping two columns in one branch silently mixes data if the types happen to match.
- `EXCEPT` and `INTERSECT` also remove duplicates; `EXCEPT ALL` and `INTERSECT ALL` keep multiplicities (PostgreSQL, DuckDB).
- MySQL gained `INTERSECT` and `EXCEPT` only in version 8.0.31; older code uses joins or `EXISTS` instead.

---

## 24. Subqueries

A **subquery** is a query nested inside another. Because every query returns a table, a subquery can appear wherever a value, a list, or a table is expected.

### Scalar subqueries: one value

A subquery that returns exactly one row and one column can be used like a single value. Products priced above the average:

```sql
SELECT name, price
FROM products
WHERE price > (SELECT avg(price) FROM products)
ORDER BY price DESC;
```

```text
     name      |  price
---------------+---------
 Desktop Tower | 1499.00
 Laptop Pro 14 | 1299.00
 Laptop Air 13 |  999.00
(3 rows)
```

Scalar subqueries also work in `SELECT`, for example to show each value next to a global figure:

```sql
SELECT name,
       price,
       round(price / (SELECT max(price) FROM products) * 100, 1) AS pct_of_max
FROM products
WHERE category_id = 3;
```

```text
     name      |  price  | pct_of_max
---------------+---------+------------
 Laptop Pro 14 | 1299.00 |       86.7
 Laptop Air 13 |  999.00 |       66.6
(2 rows)
```

If a scalar subquery returns more than one row, the query fails at runtime; if it returns none, the value is `NULL`.

### List subqueries: IN

A single-column subquery can feed `IN`. Customers who bought any book:

```sql
SELECT first_name
FROM customers
WHERE customer_id IN (
    SELECT o.customer_id
    FROM orders o
    JOIN order_items oi USING (order_id)
    JOIN products p USING (product_id)
    WHERE p.category_id = 6
)
ORDER BY first_name;
```

```text
 first_name
------------
 Alice
 Bruno
 Chloe
 Hugo
(4 rows)
```

### Correlated subqueries

A **correlated** subquery refers to the outer query's current row, so conceptually it runs once per outer row. Products priced above the average **of their own category**:

```sql
SELECT p.name, p.category_id, p.price
FROM products AS p
WHERE p.price > (
    SELECT avg(p2.price)
    FROM products AS p2
    WHERE p2.category_id = p.category_id     -- correlation: refers to the outer row
)
ORDER BY p.category_id, p.price DESC;
```

```text
            name             | category_id |  price
-----------------------------+-------------+---------
 Desktop Tower               |           2 | 1499.00
 Laptop Pro 14               |           3 | 1299.00
 Noise-Cancelling Headphones |           4 |  199.00
 Mechanical Keyboard         |           4 |   89.90
 Designing Data Systems      |           6 |   55.00
 Python Deep Dive            |           6 |   49.00
(6 rows)
```

The planner often rewrites correlated subqueries into joins, but not always. On large tables, a window function ([Section 27](#27-window-function-basics)) or a pre-aggregated join is usually the faster formulation of the same question.

### Derived tables: subqueries in FROM

A subquery in `FROM` (a *derived table*) must have an alias. It lets you aggregate in steps, for example computing order totals first and then summarizing them, which is also the fix for the fan-out trap:

```sql
SELECT round(avg(order_total), 2) AS avg_order_value,
       max(order_total)           AS largest_order
FROM (
    SELECT order_id, sum(quantity * unit_price) AS order_total
    FROM order_items
    GROUP BY order_id
) AS totals;
```

```text
 avg_order_value | largest_order
-----------------+---------------
          546.58 |       1498.00
(1 row)
```

It is also how to filter on a computed alias, which [Section 5](#5-how-a-query-runs-written-order-vs-logical-order) said `WHERE` cannot see directly:

```sql
SELECT *
FROM (SELECT name, price * 1.24 AS price_with_vat FROM products) AS t
WHERE price_with_vat > 1000;
```

```text
     name      | price_with_vat
---------------+----------------
 Laptop Pro 14 |      1610.7600
 Laptop Air 13 |      1238.7600
 Desktop Tower |      1858.7600
(3 rows)
```

### ANY and ALL

`x > ANY (subquery)` is true if the comparison holds for at least one value; `x > ALL (subquery)` if it holds for every value. Employees paid more than everyone in Sales:

```sql
SELECT first_name, salary
FROM employees
WHERE salary > ALL (SELECT salary FROM employees WHERE department = 'Sales')
ORDER BY salary DESC;
```

```text
 first_name |  salary
------------+-----------
 Sofia      | 180000.00
 Marcus     | 150000.00
(2 rows)
```

`= ANY (...)` is equivalent to `IN`. In practice `> ALL` is usually written as `> (SELECT max(...))`, which is clearer and handles empty sets more predictably.

**Notes:**
- A scalar subquery must return at most one row. Guard against surprises with an aggregate (`max`, `min`) or `LIMIT 1` with an `ORDER BY`.
- Give every derived table an alias (`AS totals`), even if you never reference it. PostgreSQL required one before version 16, and many other databases still do.
- Correlated subqueries are easy to write and easy to make slow. If one is slow, rewrite it as a join to a grouped subquery or as a window function.
- Deeply nested subqueries are hard to read. When nesting passes two levels, switch to CTEs ([Section 25](#25-common-table-expressions)).

---

## 25. Common Table Expressions

A **common table expression** (CTE) is a named subquery defined at the top of a query with `WITH`. It does nothing a derived table cannot do, but it lets you write a complex query as a sequence of readable, named steps, top to bottom, instead of inside out.

```sql
WITH order_totals AS (
    SELECT order_id, sum(quantity * unit_price) AS total
    FROM order_items
    GROUP BY order_id
)
SELECT o.customer_id, count(*) AS orders, sum(t.total) AS lifetime_value
FROM orders AS o
JOIN order_totals AS t USING (order_id)
WHERE o.status <> 'cancelled'
GROUP BY o.customer_id
ORDER BY lifetime_value DESC
LIMIT 3;
```

```text
 customer_id | orders | lifetime_value
-------------+--------+----------------
           3 |      3 |        2695.98
           4 |      2 |        1497.00
           1 |      3 |        1461.89
(3 rows)
```

### Chaining CTEs

Multiple CTEs are separated by commas, and each can refer to the ones before it. This is how analysts build multi-step logic — each step has a name and a single job, and each can be tested on its own by selecting from it.

```sql
WITH order_totals AS (          -- step 1: one row per order
    SELECT order_id, sum(quantity * unit_price) AS total
    FROM order_items
    GROUP BY order_id
),
customer_value AS (             -- step 2: one row per customer
    SELECT o.customer_id, sum(t.total) AS lifetime_value
    FROM orders o
    JOIN order_totals t USING (order_id)
    WHERE o.status <> 'cancelled'
    GROUP BY o.customer_id
),
segmented AS (                  -- step 3: label each customer
    SELECT customer_id,
           lifetime_value,
           CASE WHEN lifetime_value >= 1400 THEN 'high'
                WHEN lifetime_value >= 250  THEN 'medium'
                ELSE 'low' END AS segment
    FROM customer_value
)
SELECT segment, count(*) AS customers, sum(lifetime_value) AS value
FROM segmented
GROUP BY segment
ORDER BY value DESC;
```

```text
 segment | customers |  value
---------+-----------+---------
 high    |         3 | 5654.87
 medium  |         2 | 1682.90
 low     |         3 |  661.87
(3 rows)
```

### CTEs and performance

Since PostgreSQL 12, a CTE that is referenced once is inlined into the main query, so the planner optimizes it exactly like a subquery. A CTE referenced more than once is computed once and reused. You can force either behavior with `AS MATERIALIZED` or `AS NOT MATERIALIZED`. Before version 12, every CTE was an "optimization fence", which is why older advice warns against them. Other databases differ, but in all of them readability is the main reason to use CTEs.

### Data-modifying CTEs

In PostgreSQL, a CTE can contain `INSERT`, `UPDATE`, or `DELETE` with `RETURNING`, so one statement can, for example, archive and delete rows atomically. This is covered with `RETURNING` in [Section 32](#32-insert).

**Notes:**
- Name CTEs after what one row represents (`order_totals`, `customer_value`), which makes the grain of each step obvious and prevents fan-out bugs.
- To debug a long CTE chain, temporarily replace the final `SELECT` with `SELECT * FROM step_2` and inspect the intermediate result.
- CTEs are standard SQL and available in every modern engine (MySQL since 8.0). They are the foundation of dbt models and most analytics code.
- A CTE exists only for the single statement it is attached to. For a reusable definition, create a view ([Section 38](#38-views-and-materialized-views)).

---

## 26. Recursive CTEs

A **recursive CTE** refers to itself, which lets SQL walk hierarchies and graphs of unknown depth — category trees, org charts, bills of materials, and dependency chains.

It has two parts joined by `UNION ALL` — an **anchor** query that produces the starting rows, and a **recursive** query that joins the CTE to the table to produce the next level. The database repeats the recursive part, feeding it only the rows produced by the previous iteration, until it produces no new rows.

```sql
WITH RECURSIVE counter AS (
    SELECT 1 AS n                 -- anchor
    UNION ALL
    SELECT n + 1 FROM counter     -- recursive step
    WHERE n < 5                   -- termination condition
)
SELECT n FROM counter;
```

```text
 n
---
 1
 2
 3
 4
 5
(5 rows)
```

### Walking a tree: the category hierarchy

Starting from the root categories (no parent), each iteration finds the children of the previous level, building a path and a depth along the way:

```sql
WITH RECURSIVE tree AS (
    SELECT category_id, name, parent_id,
           0            AS depth,
           name::text   AS path
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    SELECT c.category_id, c.name, c.parent_id,
           t.depth + 1,
           t.path || ' > ' || c.name
    FROM categories AS c
    JOIN tree AS t ON c.parent_id = t.category_id
)
SELECT repeat('    ', depth) || name AS category, depth, path
FROM tree
ORDER BY path;
```

```text
    category     | depth |               path
-----------------+-------+-----------------------------------
 Books           |     0 | Books
     Programming |     1 | Books > Programming
 Electronics     |     0 | Electronics
     Accessories |     1 | Electronics > Accessories
     Computers   |     1 | Electronics > Computers
         Laptops |     2 | Electronics > Computers > Laptops
 Home            |     0 | Home
(7 rows)
```

### An org chart: everyone under a manager

All employees reporting to the CTO, directly or indirectly, with their level below him:

```sql
WITH RECURSIVE reports AS (
    SELECT employee_id, first_name, manager_id, 0 AS level
    FROM employees
    WHERE employee_id = 2          -- start at the CTO

    UNION ALL

    SELECT e.employee_id, e.first_name, e.manager_id, r.level + 1
    FROM employees AS e
    JOIN reports AS r ON e.manager_id = r.employee_id
)
SELECT * FROM reports ORDER BY level, employee_id;
```

```text
 employee_id | first_name | manager_id | level
-------------+------------+------------+-------
           2 | Marcus     |          1 |     0
           4 | Leo        |          2 |     1
           5 | Mia        |          2 |     1
           6 | Omar       |          4 |     2
(4 rows)
```

Walking **up** the tree is the same pattern in reverse: start at one node and repeatedly join to its parent. For example, every ancestor of the `Laptops` category:

```sql
WITH RECURSIVE ancestors AS (
    SELECT category_id, name, parent_id FROM categories WHERE name = 'Laptops'
    UNION ALL
    SELECT c.category_id, c.name, c.parent_id
    FROM categories c JOIN ancestors a ON c.category_id = a.parent_id
)
SELECT name FROM ancestors;
```

```text
    name
-------------
 Laptops
 Computers
 Electronics
(3 rows)
```

### Guarding against cycles

If the data contains a cycle — A is B's parent and B is A's parent — a recursive CTE loops forever. Protect real-world queries with a depth limit (`WHERE t.depth < 20`) or by tracking visited nodes in an array. PostgreSQL 14 added the standard `CYCLE` clause, which does the tracking for you.

**Notes:**
- The anchor and recursive parts are joined with `UNION ALL` (or `UNION`, which also stops on repeated rows at the cost of deduplication).
- The recursive part may reference the CTE only once, and cannot use aggregates or `LIMIT` on it directly.
- Recursive CTEs also generate series (dates, numbers) in databases that lack `generate_series`, such as MySQL and SQL Server before 2022.
- `WITH RECURSIVE` is the spelling in PostgreSQL, MySQL, SQLite, and DuckDB. SQL Server and Oracle use plain `WITH` for recursive CTEs too.
- Hierarchy questions — org charts, category paths, "all descendants of X" — are a staple of senior SQL interviews. The anchor-plus-recursive-join shape is always the answer.

---

## 27. Window Function Basics

A **window function** computes a value for each row using a set of related rows (its *window*), **without collapsing the rows**. `GROUP BY` turns nine employees into four department rows; a window function keeps all nine employees and adds the department figure to each. This one idea unlocks rankings, running totals, period-over-period comparisons, and most of analytical SQL.

The syntax is a function followed by `OVER (...)`:

```text
function(args) OVER (
    PARTITION BY ...   -- split rows into independent groups (optional)
    ORDER BY ...       -- order rows within each group (optional)
    frame clause       -- which rows around the current one to use (optional, Section 29)
)
```

### An empty window: the whole result

`OVER ()` makes the window every row of the result, so each row can be compared with a global figure:

```sql
SELECT first_name,
       salary,
       avg(salary) OVER ()                          AS company_avg,
       salary - avg(salary) OVER ()                 AS diff_from_avg,
       round(100.0 * salary / sum(salary) OVER (), 1) AS pct_of_payroll
FROM employees
ORDER BY salary DESC;
```

```text
 first_name |  salary   |     company_avg     |    diff_from_avg    | pct_of_payroll
------------+-----------+---------------------+---------------------+----------------
 Sofia      | 180000.00 | 104444.444444444444 |  75555.555555555556 |           19.1
 Marcus     | 150000.00 | 104444.444444444444 |  45555.555555555556 |           16.0
 Priya      | 120000.00 | 104444.444444444444 |  15555.555555555556 |           12.8
 Leo        | 110000.00 | 104444.444444444444 |   5555.555555555556 |           11.7
 Mia        |  95000.00 | 104444.444444444444 |  -9444.444444444444 |           10.1
 Omar       |  95000.00 | 104444.444444444444 |  -9444.444444444444 |           10.1
 Eva        |  70000.00 | 104444.444444444444 | -34444.444444444444 |            7.4
 Nina       |  60000.00 | 104444.444444444444 | -44444.444444444444 |            6.4
 Tom        |  60000.00 | 104444.444444444444 | -44444.444444444444 |            6.4
(9 rows)
```

Compare this with a scalar subquery ([Section 24](#24-subqueries)) — same result, but the window version reads the table once and is easier to extend.

### PARTITION BY: per-group figures

`PARTITION BY` splits the rows into groups, and the function is computed separately within each. Every employee next to their department's average and headcount:

```sql
SELECT first_name,
       department,
       salary,
       round(avg(salary) OVER (PARTITION BY department), 0) AS dept_avg,
       count(*)          OVER (PARTITION BY department)     AS dept_size,
       max(salary)       OVER (PARTITION BY department)     AS dept_max
FROM employees
ORDER BY department, salary DESC;
```

```text
 first_name | department  |  salary   | dept_avg | dept_size | dept_max
------------+-------------+-----------+----------+-----------+-----------
 Marcus     | Engineering | 150000.00 |   112500 |         4 | 150000.00
 Leo        | Engineering | 110000.00 |   112500 |         4 | 150000.00
 Mia        | Engineering |  95000.00 |   112500 |         4 | 150000.00
 Omar       | Engineering |  95000.00 |   112500 |         4 | 150000.00
 Sofia      | Executive   | 180000.00 |   180000 |         1 | 180000.00
 Priya      | Sales       | 120000.00 |    80000 |         3 | 120000.00
 Nina       | Sales       |  60000.00 |    80000 |         3 | 120000.00
 Tom        | Sales       |  60000.00 |    80000 |         3 | 120000.00
 Eva        | Support     |  70000.00 |    70000 |         1 |  70000.00
(9 rows)
```

Think of `PARTITION BY` as `GROUP BY` that does not collapse rows.

### ORDER BY inside the window: cumulative figures

Adding `ORDER BY` inside `OVER` changes the meaning: an aggregate then covers the rows **from the start of the partition up to the current row**, which gives running totals:

```sql
SELECT order_id,
       ordered_at::date,
       customer_id,
       count(*) OVER (ORDER BY ordered_at)                            AS orders_so_far,
       count(*) OVER (PARTITION BY customer_id ORDER BY ordered_at)   AS nth_order_for_customer
FROM orders
WHERE customer_id IN (1, 3)
ORDER BY ordered_at;
```

```text
 order_id | ordered_at | customer_id | orders_so_far | nth_order_for_customer
----------+------------+-------------+---------------+------------------------
        1 | 2026-01-05 |           1 |             1 |                      1
        3 | 2026-01-28 |           1 |             2 |                      2
        4 | 2026-02-03 |           3 |             3 |                      1
        7 | 2026-03-09 |           3 |             4 |                      2
        9 | 2026-03-30 |           1 |             5 |                      3
       15 | 2026-06-20 |           3 |             6 |                      3
(6 rows)
```

(`orders_so_far` counted only the rows that survived the `WHERE`, because window functions run after `WHERE`.)

### Where window functions can be used

Window functions are computed in the `SELECT` step, after `WHERE`, `GROUP BY`, and `HAVING` ([Section 5](#5-how-a-query-runs-written-order-vs-logical-order)). So they may appear only in `SELECT` and `ORDER BY`, and you **cannot filter on them directly**:

```sql
SELECT first_name, salary, rank() OVER (ORDER BY salary DESC) AS r
FROM employees
WHERE rank() OVER (ORDER BY salary DESC) <= 3;
```

```text
ERROR:  window functions are not allowed in WHERE
LINE 3: WHERE rank() OVER (ORDER BY salary DESC) <= 3;
              ^
```

Wrap the query in a CTE or subquery, then filter the outer query:

```sql
WITH ranked AS (
    SELECT first_name, salary, rank() OVER (ORDER BY salary DESC) AS r
    FROM employees
)
SELECT * FROM ranked WHERE r <= 3;
```

```text
 first_name |  salary   | r
------------+-----------+---
 Sofia      | 180000.00 | 1
 Marcus     | 150000.00 | 2
 Priya      | 120000.00 | 3
(3 rows)
```

Snowflake, BigQuery, Databricks, and DuckDB add a `QUALIFY` clause that filters on window results directly (`... QUALIFY r <= 3`). PostgreSQL does not have it.

### Window functions over grouped results

Because windows run after `GROUP BY`, they can operate on aggregates. Monthly revenue and each month's share of the total:

```sql
SELECT date_trunc('month', o.ordered_at)::date                        AS month,
       sum(oi.quantity * oi.unit_price)                                AS revenue,
       round(100.0 * sum(oi.quantity * oi.unit_price)
             / sum(sum(oi.quantity * oi.unit_price)) OVER (), 1)       AS pct_of_total
FROM orders o
JOIN order_items oi USING (order_id)
WHERE o.status <> 'cancelled'
GROUP BY month
ORDER BY month;
```

```text
   month    | revenue | pct_of_total
------------+---------+--------------
 2026-01-01 | 1445.89 |         18.1
 2026-02-01 | 1601.98 |         20.0
 2026-03-01 |  436.00 |          5.5
 2026-04-01 | 1478.87 |         18.5
 2026-05-01 | 1293.00 |         16.2
 2026-06-01 | 1743.90 |         21.8
(6 rows)
```

`sum(sum(...)) OVER ()` looks odd but is logical — the inner `sum` is the per-month aggregate from `GROUP BY`, and the outer `sum ... OVER ()` adds those monthly values across all rows.

### Named windows

When several functions share a window definition, name it once with a `WINDOW` clause:

```sql
SELECT first_name, department, salary,
       rank()     OVER w AS rnk,
       avg(salary) OVER w AS running_avg
FROM employees
WINDOW w AS (PARTITION BY department ORDER BY salary DESC)
ORDER BY department, rnk;
```

```text
 first_name | department  |  salary   | rnk |     running_avg
------------+-------------+-----------+-----+---------------------
 Marcus     | Engineering | 150000.00 |   1 | 150000.000000000000
 Leo        | Engineering | 110000.00 |   2 | 130000.000000000000
 Mia        | Engineering |  95000.00 |   3 | 112500.000000000000
 Omar       | Engineering |  95000.00 |   3 | 112500.000000000000
 Sofia      | Executive   | 180000.00 |   1 | 180000.000000000000
 Priya      | Sales       | 120000.00 |   1 | 120000.000000000000
 Nina       | Sales       |  60000.00 |   2 |  80000.000000000000
 Tom        | Sales       |  60000.00 |   2 |  80000.000000000000
 Eva        | Support     |  70000.00 |   1 |  70000.000000000000
(9 rows)
```

**Notes:**
- `GROUP BY` returns one row per group; a window function returns one value per row. If you need both detail and group figures on the same row, you need a window.
- Window functions cannot appear in `WHERE`, `GROUP BY`, or `HAVING`. Filter on them from an outer query, or with `QUALIFY` where supported.
- `ORDER BY` inside `OVER` changes aggregates from "whole partition" to "running up to this row". Forgetting this is a frequent source of confusing results ([Section 29](#29-frames-running-totals-and-moving-averages)).
- Window functions are standard (SQL:2003) and supported everywhere modern: PostgreSQL, MySQL 8+, SQLite 3.25+, SQL Server, Oracle, and every warehouse.
- Window functions are the single most tested advanced topic in data analyst and data scientist interviews.

---

## 28. Ranking and Offset Functions

Some window functions exist only as window functions. They fall into two families: **ranking** (where does this row stand?) and **offset** (what is in a neighboring row?).

### ROW_NUMBER, RANK, DENSE_RANK

All three number rows by the window's `ORDER BY`. They differ only in how they treat ties, and the difference is a classic interview question:

| Function | Ties | Example for values 100, 90, 90, 80 |
|:--|:--|:--|
| `row_number()` | Arbitrary distinct numbers | 1, 2, 3, 4 |
| `rank()` | Same rank, then a gap | 1, 2, 2, 4 |
| `dense_rank()` | Same rank, no gap | 1, 2, 2, 3 |

```sql
SELECT first_name,
       salary,
       row_number() OVER (ORDER BY salary DESC) AS row_num,
       rank()       OVER (ORDER BY salary DESC) AS rnk,
       dense_rank() OVER (ORDER BY salary DESC) AS dense_rnk
FROM employees
ORDER BY salary DESC, first_name;
```

```text
 first_name |  salary   | row_num | rnk | dense_rnk
------------+-----------+---------+-----+-----------
 Sofia      | 180000.00 |       1 |   1 |         1
 Marcus     | 150000.00 |       2 |   2 |         2
 Priya      | 120000.00 |       3 |   3 |         3
 Leo        | 110000.00 |       4 |   4 |         4
 Mia        |  95000.00 |       5 |   5 |         5
 Omar       |  95000.00 |       6 |   5 |         5
 Eva        |  70000.00 |       7 |   7 |         6
 Nina       |  60000.00 |       8 |   8 |         7
 Tom        |  60000.00 |       9 |   8 |         7
(9 rows)
```

Mia and Omar tie at 95,000: `rank` gives both 5 and skips 6; `dense_rank` gives both 5 and continues at 6; `row_number` breaks the tie arbitrarily (add a tiebreaker column to the window's `ORDER BY` to make it deterministic).

Rankings restart in each partition. Rank within department:

```sql
SELECT department, first_name, salary,
       dense_rank() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees
ORDER BY department, dept_rank;
```

```text
 department  | first_name |  salary   | dept_rank
-------------+------------+-----------+-----------
 Engineering | Marcus     | 150000.00 |         1
 Engineering | Leo        | 110000.00 |         2
 Engineering | Mia        |  95000.00 |         3
 Engineering | Omar       |  95000.00 |         3
 Executive   | Sofia      | 180000.00 |         1
 Sales       | Priya      | 120000.00 |         1
 Sales       | Nina       |  60000.00 |         2
 Sales       | Tom        |  60000.00 |         2
 Support     | Eva        |  70000.00 |         1
(9 rows)
```

### NTILE, PERCENT_RANK, CUME_DIST

`ntile(n)` splits the ordered rows into `n` nearly equal buckets (quartiles, deciles). `percent_rank` and `cume_dist` give relative standing between 0 and 1:

```sql
SELECT name, price,
       ntile(4)       OVER (ORDER BY price)                AS quartile,
       round(percent_rank() OVER (ORDER BY price)::numeric, 2) AS pct_rank,
       round(cume_dist()    OVER (ORDER BY price)::numeric, 2) AS cume_dist
FROM products
ORDER BY price;
```

```text
            name             |  price  | quartile | pct_rank | cume_dist
-----------------------------+---------+----------+----------+-----------
 Wireless Mouse              |   29.99 |        1 |     0.00 |      0.08
 SQL for Everyone            |   39.00 |        1 |     0.09 |      0.17
 USB-C Hub                   |   45.00 |        1 |     0.18 |      0.25
 Python Deep Dive            |   49.00 |        2 |     0.27 |      0.33
 Designing Data Systems      |   55.00 |        2 |     0.36 |      0.42
 Webcam HD                   |   59.00 |        2 |     0.45 |      0.50
 Mechanical Keyboard         |   89.90 |        3 |     0.55 |      0.58
 Noise-Cancelling Headphones |  199.00 |        3 |     0.64 |      0.67
 27in Monitor                |  249.00 |        3 |     0.73 |      0.75
 Laptop Air 13               |  999.00 |        4 |     0.82 |      0.83
 Laptop Pro 14               | 1299.00 |        4 |     0.91 |      0.92
 Desktop Tower               | 1499.00 |        4 |     1.00 |      1.00
(12 rows)
```

### LAG and LEAD: previous and next rows

`lag(expr, n, default)` reads a value from `n` rows **before** the current row in the window; `lead` reads **after**. They make row-to-row comparisons trivial, such as the gap between each customer's consecutive orders:

```sql
SELECT customer_id,
       order_id,
       ordered_at::date                                                        AS order_date,
       lag(ordered_at::date) OVER (PARTITION BY customer_id ORDER BY ordered_at) AS previous_order,
       ordered_at::date
         - lag(ordered_at::date) OVER (PARTITION BY customer_id ORDER BY ordered_at) AS days_since_previous
FROM orders
WHERE customer_id IN (1, 3)
ORDER BY customer_id, ordered_at;
```

```text
 customer_id | order_id | order_date | previous_order | days_since_previous
-------------+----------+------------+----------------+---------------------
           1 |        1 | 2026-01-05 | NULL           |                NULL
           1 |        3 | 2026-01-28 | 2026-01-05     |                  23
           1 |        9 | 2026-03-30 | 2026-01-28     |                  61
           3 |        4 | 2026-02-03 | NULL           |                NULL
           3 |        7 | 2026-03-09 | 2026-02-03     |                  34
           3 |       15 | 2026-06-20 | 2026-03-09     |                 103
(6 rows)
```

The first order per customer has no previous row, so `lag` returns `NULL` — or the third argument, if you supply one. `lead` answers "what happened next?", for example "time until the customer's next order", which is the basis of churn analysis.

### FIRST_VALUE, LAST_VALUE, NTH_VALUE

These return a value from a specific position in the window. Each employee next to the top earner in their department:

```sql
SELECT department, first_name, salary,
       first_value(first_name) OVER (PARTITION BY department ORDER BY salary DESC) AS top_earner
FROM employees
ORDER BY department, salary DESC;
```

```text
 department  | first_name |  salary   | top_earner
-------------+------------+-----------+------------
 Engineering | Marcus     | 150000.00 | Marcus
 Engineering | Leo        | 110000.00 | Marcus
 Engineering | Mia        |  95000.00 | Marcus
 Engineering | Omar       |  95000.00 | Marcus
 Executive   | Sofia      | 180000.00 | Sofia
 Sales       | Priya      | 120000.00 | Priya
 Sales       | Nina       |  60000.00 | Priya
 Sales       | Tom        |  60000.00 | Priya
 Support     | Eva        |  70000.00 | Eva
(9 rows)
```

`last_value` has a famous trap: with an `ORDER BY` in the window, the default frame ends at the **current row** (including rows tied with it), so `last_value` returns the current row, or the last row tied with it. The fix is an explicit frame, explained in the next section:

```sql
SELECT department, first_name, salary,
       last_value(first_name) OVER (PARTITION BY department ORDER BY salary DESC)          AS wrong_lowest,
       last_value(first_name) OVER (PARTITION BY department ORDER BY salary DESC
                                    ROWS BETWEEN UNBOUNDED PRECEDING
                                             AND UNBOUNDED FOLLOWING)                      AS lowest_earner
FROM employees
WHERE department IN ('Engineering', 'Sales')
ORDER BY department, salary DESC;
```

```text
 department  | first_name |  salary   | wrong_lowest | lowest_earner
-------------+------------+-----------+--------------+---------------
 Engineering | Marcus     | 150000.00 | Marcus       | Omar
 Engineering | Leo        | 110000.00 | Leo          | Omar
 Engineering | Mia        |  95000.00 | Omar         | Omar
 Engineering | Omar       |  95000.00 | Omar         | Omar
 Sales       | Priya      | 120000.00 | Priya        | Nina
 Sales       | Tom        |  60000.00 | Nina         | Nina
 Sales       | Nina       |  60000.00 | Nina         | Nina
(7 rows)
```

**Notes:**
- `ROW_NUMBER` vs `RANK` vs `DENSE_RANK` is asked in almost every SQL interview. Use `dense_rank` for "Nth highest distinct value", `row_number` for "exactly one row per group".
- Make `row_number()` deterministic with a unique tiebreaker in `ORDER BY` (`ORDER BY salary DESC, employee_id`), or reruns can return different rows.
- `lag` and `lead` default to `NULL` at partition edges. Pass a third argument to substitute a default: `lag(x, 1, 0)`.
- To get the last value in a partition, use `first_value` with the order reversed, or `last_value` with a full frame. Never rely on the default frame for `last_value`.

---

## 29. Frames, Running Totals, and Moving Averages

The **frame** is the subset of the partition that an aggregate window function actually uses for the current row. Frames are what turn `sum` into a running total or `avg` into a moving average.

```text
ROWS BETWEEN <start> AND <end>

start/end can be:
  UNBOUNDED PRECEDING   the first row of the partition
  n PRECEDING           n rows before the current row
  CURRENT ROW           the current row
  n FOLLOWING           n rows after the current row
  UNBOUNDED FOLLOWING   the last row of the partition
```

### The default frames

| Window has | Default frame |
|:--|:--|
| No `ORDER BY` | The whole partition |
| An `ORDER BY` | `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` (start of partition up to the current row, **including ties**) |

The "including ties" part is the subtle bit. `RANGE` frames treat all rows with the same `ORDER BY` value as peers, so tied rows are added together:

```sql
SELECT first_name, salary,
       sum(salary) OVER (ORDER BY salary)                                         AS range_default,
       sum(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS rows_frame
FROM employees
ORDER BY salary, first_name;
```

```text
 first_name |  salary   | range_default | rows_frame
------------+-----------+---------------+------------
 Nina       |  60000.00 |     120000.00 |   60000.00
 Tom        |  60000.00 |     120000.00 |  120000.00
 Eva        |  70000.00 |     190000.00 |  190000.00
 Mia        |  95000.00 |     380000.00 |  285000.00
 Omar       |  95000.00 |     380000.00 |  380000.00
 Leo        | 110000.00 |     490000.00 |  490000.00
 Priya      | 120000.00 |     610000.00 |  610000.00
 Marcus     | 150000.00 |     760000.00 |  760000.00
 Sofia      | 180000.00 |     940000.00 |  940000.00
(9 rows)
```

With `RANGE`, both 60,000 earners show 120,000 because they are peers. With `ROWS`, the running total grows row by row. For running totals, write `ROWS` explicitly unless you want peer grouping.

### Running totals

Cumulative revenue by day, and cumulative share of total revenue:

```sql
WITH daily AS (
    SELECT o.ordered_at::date AS day, sum(oi.quantity * oi.unit_price) AS revenue
    FROM orders o JOIN order_items oi USING (order_id)
    WHERE o.status <> 'cancelled'
    GROUP BY day
)
SELECT day,
       revenue,
       sum(revenue) OVER (ORDER BY day ROWS UNBOUNDED PRECEDING)                        AS running_total,
       round(100.0 * sum(revenue) OVER (ORDER BY day ROWS UNBOUNDED PRECEDING)
             / sum(revenue) OVER (), 1)                                                AS running_pct
FROM daily
ORDER BY day;
```

```text
    day     | revenue | running_total | running_pct
------------+---------+---------------+-------------
 2026-01-05 | 1228.99 |       1228.99 |        15.4
 2026-01-12 |  127.00 |       1355.99 |        17.0
 2026-01-28 |   89.90 |       1445.89 |        18.1
 2026-02-03 | 1103.98 |       2549.87 |        31.9
 2026-02-14 |  498.00 |       3047.87 |        38.1
 2026-03-09 |   94.00 |       3141.87 |        39.3
 2026-03-15 |  199.00 |       3340.87 |        41.8
 2026-03-30 |  143.00 |       3483.87 |        43.6
 2026-04-11 | 1388.90 |       4872.77 |        60.9
 2026-04-25 |   89.97 |       4962.74 |        62.0
 2026-05-06 |  294.00 |       5256.74 |        65.7
 2026-05-19 |  999.00 |       6255.74 |        78.2
 2026-06-02 |  245.90 |       6501.64 |        81.3
 2026-06-20 | 1498.00 |       7999.64 |       100.0
(14 rows)
```

(`ROWS UNBOUNDED PRECEDING` is shorthand for `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.)

### Moving averages

A moving average smooths noise by averaging each row with its neighbors. A three-order moving average of order value:

```sql
WITH order_totals AS (
    SELECT o.order_id, o.ordered_at::date AS day, sum(oi.quantity * oi.unit_price) AS total
    FROM orders o JOIN order_items oi USING (order_id)
    GROUP BY o.order_id, day
)
SELECT order_id, day, total,
       round(avg(total) OVER (ORDER BY day ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS moving_avg_3,
       count(*)         OVER (ORDER BY day ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)     AS rows_in_frame
FROM order_totals
ORDER BY day;
```

```text
 order_id |    day     |  total  | moving_avg_3 | rows_in_frame
----------+------------+---------+--------------+---------------
        1 | 2026-01-05 | 1228.99 |      1228.99 |             1
        2 | 2026-01-12 |  127.00 |       678.00 |             2
        3 | 2026-01-28 |   89.90 |       481.96 |             3
        4 | 2026-02-03 | 1103.98 |       440.29 |             3
        5 | 2026-02-14 |  498.00 |       563.96 |             3
        6 | 2026-02-27 |  199.00 |       600.33 |             3
        7 | 2026-03-09 |   94.00 |       263.67 |             3
        8 | 2026-03-15 |  199.00 |       164.00 |             3
        9 | 2026-03-30 |  143.00 |       145.33 |             3
       10 | 2026-04-11 | 1388.90 |       576.97 |             3
       11 | 2026-04-25 |   89.97 |       540.62 |             3
       12 | 2026-05-06 |  294.00 |       590.96 |             3
       13 | 2026-05-19 |  999.00 |       460.99 |             3
       14 | 2026-06-02 |  245.90 |       512.97 |             3
       15 | 2026-06-20 | 1498.00 |       914.30 |             3
(15 rows)
```

The first two rows average fewer than three values, visible in `rows_in_frame`. Filter them out or show them as `NULL` if a partial window would mislead.

### Time-based frames with RANGE

`ROWS` counts rows; `RANGE` with an interval uses the **values** in the `ORDER BY` column, so "the last 30 days" means 30 calendar days no matter how many rows fall in them:

```sql
SELECT order_id, ordered_at::date AS day,
       count(*) OVER (ORDER BY ordered_at
                      RANGE BETWEEN interval '30 days' PRECEDING AND CURRENT ROW) AS orders_last_30_days
FROM orders
ORDER BY ordered_at;
```

```text
 order_id |    day     | orders_last_30_days
----------+------------+---------------------
        1 | 2026-01-05 |                   1
        2 | 2026-01-12 |                   2
        3 | 2026-01-28 |                   3
        4 | 2026-02-03 |                   4
        5 | 2026-02-14 |                   3
        6 | 2026-02-27 |                   4
        7 | 2026-03-09 |                   3
        8 | 2026-03-15 |                   4
        9 | 2026-03-30 |                   3
       10 | 2026-04-11 |                   3
       11 | 2026-04-25 |                   3
       12 | 2026-05-06 |                   3
       13 | 2026-05-19 |                   3
       14 | 2026-06-02 |                   3
       15 | 2026-06-20 |                   2
(15 rows)
```

This is the correct tool for rolling time windows on irregular data, where a `ROWS` frame would silently mix different time spans.

**Notes:**
- The default frame with `ORDER BY` is `RANGE ... CURRENT ROW`, which groups ties. Write `ROWS` explicitly for running totals and moving averages.
- A moving average over rows assumes evenly spaced data. With gaps (missing days), either fill the gaps with a date spine first ([Section 31](#31-analytics-playbook-cohorts-retention-funnels-growth)) or use a `RANGE` interval frame.
- `GROUPS` frames (PostgreSQL 11+) count peer groups instead of rows, a rarely needed third option.
- Frames apply only to aggregate and value functions (`sum`, `avg`, `first_value`, and so on). Ranking functions and `lag`/`lead` ignore them.

---

## 30. Classic Patterns: Top-N, Dedup, Gaps and Islands, Pivots

These patterns appear constantly in real work and in interviews. Each is a combination of techniques you have already seen.

### Top N per group

"The two best-selling products in each category" cannot be done with `LIMIT`, which applies to the whole result. Rank within each group, then filter:

```sql
WITH product_sales AS (
    SELECT p.category_id, p.name, sum(oi.quantity) AS units
    FROM products p
    JOIN order_items oi USING (product_id)
    GROUP BY p.category_id, p.name
),
ranked AS (
    SELECT *, row_number() OVER (PARTITION BY category_id ORDER BY units DESC, name) AS rn
    FROM product_sales
)
SELECT category_id, name, units
FROM ranked
WHERE rn <= 2
ORDER BY category_id, rn;
```

```text
 category_id |          name          | units
-------------+------------------------+-------
           2 | 27in Monitor           |     3
           3 | Laptop Pro 14          |     3
           3 | Laptop Air 13          |     2
           4 | Wireless Mouse         |     6
           4 | Mechanical Keyboard    |     3
           6 | SQL for Everyone       |     8
           6 | Designing Data Systems |     2
(7 rows)
```

Use `rank` or `dense_rank` instead of `row_number` if tied items should all be included.

### Nth highest value

"Find the second highest salary" is probably the most asked SQL interview question. `dense_rank` handles ties correctly:

```sql
WITH ranked AS (
    SELECT DISTINCT salary, dense_rank() OVER (ORDER BY salary DESC) AS dr
    FROM employees
)
SELECT salary FROM ranked WHERE dr = 2;
```

```text
  salary
-----------
 150000.00
(1 row)
```

The classic non-window answer, `SELECT max(salary) FROM employees WHERE salary < (SELECT max(salary) FROM employees)`, works for the second highest but does not generalize to the Nth. `ORDER BY salary DESC LIMIT 1 OFFSET 1` is wrong when the top salary is tied, unless you add `DISTINCT`.

### Deduplication: keep one row per key

Duplicates arrive from double-submitted forms, retried pipeline loads, and merged sources. To keep the **most recent** row per email from a raw sign-up feed:

```sql
WITH raw_signups (email, name, loaded_at) AS (
    VALUES ('a@example.com', 'Ann',     '2026-01-01 10:00'::timestamp),
           ('a@example.com', 'Ann B.',  '2026-01-03 09:00'),
           ('b@example.com', 'Ben',     '2026-01-02 12:00'),
           ('a@example.com', 'Ann',     '2026-01-02 08:00')
),
ranked AS (
    SELECT *, row_number() OVER (PARTITION BY email ORDER BY loaded_at DESC) AS rn
    FROM raw_signups
)
SELECT email, name, loaded_at FROM ranked WHERE rn = 1 ORDER BY email;
```

```text
     email     |  name  |      loaded_at
---------------+--------+---------------------
 a@example.com | Ann B. | 2026-01-03 09:00:00
 b@example.com | Ben    | 2026-01-02 12:00:00
(2 rows)
```

To find duplicates rather than remove them, group and count: `SELECT email, count(*) FROM raw_signups GROUP BY email HAVING count(*) > 1`. To delete duplicates from a real table, see [Section 33](#33-update-delete-and-truncate).

### Gaps and islands

An **island** is a run of consecutive values; a **gap** is a break between runs. "Longest login streak", "periods a sensor was offline", and "consecutive months with a sale" are all this problem. The trick: subtract a `row_number` from the value. Within a consecutive run, both increase by one each step, so the difference stays constant and identifies the island:

```sql
WITH logins (user_id, day) AS (
    VALUES (1, DATE '2026-03-01'), (1, '2026-03-02'), (1, '2026-03-03'),
           (1, '2026-03-06'), (1, '2026-03-07'),
           (1, '2026-03-10')
),
marked AS (
    SELECT user_id, day,
           day - (row_number() OVER (PARTITION BY user_id ORDER BY day))::int AS island_id
    FROM logins
)
SELECT user_id,
       min(day)  AS streak_start,
       max(day)  AS streak_end,
       count(*)  AS streak_days
FROM marked
GROUP BY user_id, island_id
ORDER BY streak_start;
```

```text
 user_id | streak_start | streak_end | streak_days
---------+--------------+------------+-------------
       1 | 2026-03-01   | 2026-03-03 |           3
       1 | 2026-03-06   | 2026-03-07 |           2
       1 | 2026-03-10   | 2026-03-10 |           1
(3 rows)
```

A related technique finds the **gaps** directly with `lead` — any row whose next value is more than one step away marks the start of a gap.

### Pivoting: rows to columns

A pivot turns values in a column into separate columns. Conditional aggregation is the portable way, with `FILTER` or, where that is unsupported, `CASE` (each output column is written explicitly):

```sql
SELECT cat.name AS category,
       sum(oi.quantity) FILTER (WHERE o.ordered_at <  '2026-04-01') AS q1_units,
       sum(oi.quantity) FILTER (WHERE o.ordered_at >= '2026-04-01') AS q2_units,
       sum(oi.quantity)                                            AS total_units
FROM order_items oi
JOIN orders o     USING (order_id)
JOIN products p   USING (product_id)
JOIN categories cat ON cat.category_id = p.category_id
GROUP BY cat.name
ORDER BY total_units DESC;
```

```text
  category   | q1_units | q2_units | total_units
-------------+----------+----------+-------------
 Accessories |        7 |        7 |          14
 Programming |        8 |        4 |          12
 Laptops     |        2 |        3 |           5
 Computers   |        2 |        1 |           3
(4 rows)
```

SQL cannot create a dynamic number of columns from data in a single static query — the column list must be known in advance. SQL Server and Snowflake have a `PIVOT` operator, PostgreSQL has `crosstab` in the `tablefunc` extension, and DuckDB has `PIVOT`, but all of them still fix the output shape at query time. For truly dynamic pivots, pivot in the client (pandas `pivot_table`) or generate the SQL.

### Unpivoting: columns to rows

The reverse, turning columns into rows, is neatly done with `LATERAL` and `VALUES`:

```sql
SELECT p.name, x.metric, x.value
FROM products p
CROSS JOIN LATERAL (VALUES ('price', p.price), ('stock', p.stock::numeric)) AS x(metric, value)
WHERE p.product_id IN (1, 2)
ORDER BY p.name, x.metric;
```

```text
     name      | metric |  value
---------------+--------+---------
 Laptop Air 13 | price  |  999.00
 Laptop Air 13 | stock  |      20
 Laptop Pro 14 | price  | 1299.00
 Laptop Pro 14 | stock  |      15
(4 rows)
```

**Notes:**
- Top-N per group: rank in a CTE, filter in the outer query. It is the same shape every time.
- For "Nth highest", say out loud how you handle ties. Interviewers are testing whether you think of them.
- Deduplicate with `row_number() ... = 1`, choosing the `ORDER BY` that defines which duplicate wins — latest, most complete, highest priority.
- Gaps and islands: `value - row_number()` for consecutive integers or dates. For timestamps with a tolerance — sessions that end after 30 minutes of inactivity — flag gaps with `lag` and number the islands with a running `sum` of the flags.
- Pivot with `FILTER` or `CASE` for portability. Operators like `PIVOT` are convenient but dialect-specific.

---

## 31. Analytics Playbook: Cohorts, Retention, Funnels, Growth

The questions analysts and data scientists answer every week share a few standard shapes. Each is a short chain of CTEs built from techniques in the previous sections.

### Gap-free time series with a date spine

Grouping by month only produces months that have data. For charts and period-over-period math, every period must exist, even with zero activity. Generate the calendar, then left join the facts onto it:

```sql
WITH months AS (
    SELECT generate_series('2026-01-01'::date, '2026-08-01'::date, interval '1 month')::date AS month
),
monthly AS (
    SELECT date_trunc('month', ordered_at)::date AS month, count(*) AS orders
    FROM orders
    WHERE status <> 'cancelled'
    GROUP BY 1
)
SELECT m.month, COALESCE(monthly.orders, 0) AS orders
FROM months m
LEFT JOIN monthly USING (month)
ORDER BY m.month;
```

```text
   month    | orders
------------+--------
 2026-01-01 |      3
 2026-02-01 |      2
 2026-03-01 |      3
 2026-04-01 |      2
 2026-05-01 |      2
 2026-06-01 |      2
 2026-07-01 |      0
 2026-08-01 |      0
(8 rows)
```

July and August now appear as explicit zeros rather than disappearing.

### Period-over-period growth

`lag` over a monthly series gives month-over-month change; `NULLIF` protects against division by zero:

```sql
WITH monthly AS (
    SELECT date_trunc('month', o.ordered_at)::date AS month,
           sum(oi.quantity * oi.unit_price)       AS revenue
    FROM orders o JOIN order_items oi USING (order_id)
    WHERE o.status <> 'cancelled'
    GROUP BY 1
)
SELECT month,
       revenue,
       lag(revenue) OVER (ORDER BY month)                                   AS prev_revenue,
       round(100.0 * (revenue - lag(revenue) OVER (ORDER BY month))
             / NULLIF(lag(revenue) OVER (ORDER BY month), 0), 1)              AS mom_growth_pct
FROM monthly
ORDER BY month;
```

```text
   month    | revenue | prev_revenue | mom_growth_pct
------------+---------+--------------+----------------
 2026-01-01 | 1445.89 |         NULL |           NULL
 2026-02-01 | 1601.98 |      1445.89 |           10.8
 2026-03-01 |  436.00 |      1601.98 |          -72.8
 2026-04-01 | 1478.87 |       436.00 |          239.2
 2026-05-01 | 1293.00 |      1478.87 |          -12.6
 2026-06-01 | 1743.90 |      1293.00 |           34.9
(6 rows)
```

Year-over-year works the same way with `lag(revenue, 12)` on a gap-free monthly series, which is why the date spine matters: `lag(x, 12)` means "12 rows back", which is only "12 months back" if no month is missing.

### Cohort retention

A **cohort** is a group of users who share a starting period, here the month of their first order. Retention asks: of each cohort, how many ordered again 1, 2, 3 months later? This is the standard way to see whether a product keeps its customers:

```sql
WITH customer_orders AS (
    SELECT customer_id, date_trunc('month', ordered_at)::date AS order_month
    FROM orders
    WHERE status <> 'cancelled'
),
cohorts AS (
    SELECT customer_id, min(order_month) AS cohort_month
    FROM customer_orders
    GROUP BY customer_id
),
activity AS (
    SELECT c.cohort_month,
           (extract(year FROM age(co.order_month, c.cohort_month)) * 12
            + extract(month FROM age(co.order_month, c.cohort_month)))::int AS month_number,
           co.customer_id
    FROM customer_orders co
    JOIN cohorts c USING (customer_id)
)
SELECT cohort_month,
       count(DISTINCT customer_id) FILTER (WHERE month_number = 0) AS m0,
       count(DISTINCT customer_id) FILTER (WHERE month_number = 1) AS m1,
       count(DISTINCT customer_id) FILTER (WHERE month_number = 2) AS m2,
       count(DISTINCT customer_id) FILTER (WHERE month_number = 3) AS m3
FROM activity
GROUP BY cohort_month
ORDER BY cohort_month;
```

```text
 cohort_month | m0 | m1 | m2 | m3
--------------+----+----+----+----
 2026-01-01   |  2 |  0 |  1 |  1
 2026-02-01   |  2 |  1 |  0 |  1
 2026-03-01   |  1 |  0 |  0 |  0
 2026-04-01   |  1 |  0 |  0 |  0
 2026-05-01   |  1 |  0 |  0 |  0
 2026-06-01   |  1 |  0 |  0 |  0
(6 rows)
```

Read each row left to right — the January cohort had 2 customers, 1 of whom ordered again in month 2 (March) and 1 in month 3 (April). Dividing each column by `m0` turns counts into retention rates.

### Funnels

A **funnel** measures how many users make it through a sequence of steps. Given an events table, count distinct users reaching each step and the conversion from the step before:

```sql
WITH events (user_id, event) AS (
    VALUES (1,'view'),(1,'add_to_cart'),(1,'checkout'),(1,'purchase'),
           (2,'view'),(2,'add_to_cart'),
           (3,'view'),(3,'add_to_cart'),(3,'checkout'),
           (4,'view'),
           (5,'view'),(5,'add_to_cart'),(5,'checkout'),(5,'purchase')
),
steps (step_no, event) AS (
    VALUES (1,'view'),(2,'add_to_cart'),(3,'checkout'),(4,'purchase')
),
reached AS (
    SELECT s.step_no, s.event, count(DISTINCT e.user_id) AS users
    FROM steps s
    LEFT JOIN events e USING (event)
    GROUP BY s.step_no, s.event
)
SELECT step_no, event, users,
       round(100.0 * users / first_value(users) OVER (ORDER BY step_no), 1) AS pct_of_start,
       round(100.0 * users / lag(users) OVER (ORDER BY step_no), 1)         AS pct_of_previous
FROM reached
ORDER BY step_no;
```

```text
 step_no |    event    | users | pct_of_start | pct_of_previous
---------+-------------+-------+--------------+-----------------
       1 | view        |     5 |        100.0 |            NULL
       2 | add_to_cart |     4 |         80.0 |            80.0
       3 | checkout    |     3 |         60.0 |            75.0
       4 | purchase    |     2 |         40.0 |            66.7
(4 rows)
```

Real funnels usually also require the steps to happen **in order** and within a time limit, which adds a timestamp comparison per step (for example, a `checkout` counts only if it follows that user's `add_to_cart`).

### Pareto analysis: who drives the revenue?

The cumulative share of revenue by customer, sorted from largest to smallest, shows concentration (the "80/20 rule") and supports ABC segmentation:

```sql
WITH customer_revenue AS (
    SELECT o.customer_id, sum(oi.quantity * oi.unit_price) AS revenue
    FROM orders o JOIN order_items oi USING (order_id)
    WHERE o.status <> 'cancelled'
    GROUP BY o.customer_id
)
SELECT customer_id,
       revenue,
       round(100.0 * sum(revenue) OVER (ORDER BY revenue DESC ROWS UNBOUNDED PRECEDING)
             / sum(revenue) OVER (), 1)                                              AS cumulative_pct,
       CASE WHEN sum(revenue) OVER (ORDER BY revenue DESC ROWS UNBOUNDED PRECEDING)
                 - revenue < 0.8 * sum(revenue) OVER () THEN 'A' ELSE 'B/C' END      AS abc_class
FROM customer_revenue
ORDER BY revenue DESC;
```

```text
 customer_id | revenue | cumulative_pct | abc_class
-------------+---------+----------------+-----------
           3 | 2695.98 |           33.7 | A
           4 | 1497.00 |           52.4 | A
           1 | 1461.89 |           70.7 | A
           7 | 1388.90 |           88.1 | A
           5 |  294.00 |           91.7 | B/C
           8 |  245.90 |           94.8 | B/C
           2 |  216.97 |           97.5 | B/C
           6 |  199.00 |          100.0 | B/C
(8 rows)
```

Here four customers generate over 80 percent of revenue.

**Notes:**
- Build every time series on a date spine. Missing periods break charts, moving averages, and `lag`-based comparisons silently.
- Define the metric before writing SQL: does "active" mean ordered, logged in, or paid? Does revenue include cancelled orders, tax, refunds? Most analytics bugs are definition bugs, not SQL bugs.
- Cohort tables are long-to-wide pivots — many teams compute the long form (`cohort_month, month_number, customers`) in SQL and pivot in the BI tool.
- `count(DISTINCT user_id)` is the correct measure for users; `count(*)` counts events. Mixing them up inflates every funnel.

---

## 32. INSERT

`INSERT` adds rows to a table. Always name the columns you are filling — it documents intent, and it keeps working when someone adds or reorders columns.

### Single and multiple rows

```sql
INSERT INTO customers (first_name, last_name, email, city, country, signup_date)
VALUES ('Kai', 'Tanaka', 'kai@example.com', 'Osaka', 'Japan', '2026-07-01');
```

```text
INSERT 0 1
```

The output `INSERT 0 1` means one row was inserted (the `0` is a historical leftover). `customer_id` was not listed, so its identity column generated the next value automatically. One statement can insert many rows, which is much faster than many single-row statements:

```sql
INSERT INTO categories (name, parent_id)
VALUES ('Kitchen', 7),
       ('Garden',  7);
```

```text
INSERT 0 2
```

### Defaults

Omitted columns get their `DEFAULT` value, or `NULL` if there is none. The keyword `DEFAULT` requests it explicitly:

```sql
INSERT INTO products (name, category_id, price, stock)
VALUES ('Desk Lamp', 7, 34.50, DEFAULT);

SELECT product_id, name, stock, attributes FROM products WHERE name = 'Desk Lamp';
```

```text
INSERT 0 1
 product_id |   name    | stock | attributes
------------+-----------+-------+------------
         13 | Desk Lamp |     0 | NULL
(1 row)
```

### Constraints guard every insert

Every constraint is checked on every row. A violation rejects the whole statement, so a multi-row insert with one bad row inserts nothing:

```sql
INSERT INTO customers (first_name, last_name, email, country, signup_date)
VALUES ('Lena', 'Fox', 'lena@example.com', 'Austria', '2026-07-02'),
       ('Alice', 'Copy', 'alice@example.com', 'UK', '2026-07-02');   -- duplicate email

SELECT count(*) FROM customers WHERE first_name = 'Lena';
```

```text
ERROR:  duplicate key value violates unique constraint "customers_email_key"
DETAIL:  Key (email)=(alice@example.com) already exists.
 count
-------
     0
(1 row)
```

### RETURNING

PostgreSQL's `RETURNING` clause hands back values from the rows just written, most importantly generated keys, so the application does not need a second query:

```sql
INSERT INTO orders (customer_id, employee_id, ordered_at, status)
VALUES (9, 7, '2026-07-03 10:00+00', 'pending')
RETURNING order_id, status;
```

```text
 order_id | status
----------+---------
       16 | pending
(1 row)

INSERT 0 1
```

`RETURNING` also works with `UPDATE` and `DELETE`. SQL Server has an equivalent `OUTPUT` clause; MySQL has neither and uses `LAST_INSERT_ID()`.

### INSERT ... SELECT

The source of an insert can be any query. This is how data moves between tables in pipelines: staging to production, raw to cleaned, detail to summary:

```sql
CREATE TABLE monthly_revenue (
    month   date PRIMARY KEY,
    revenue numeric(12,2) NOT NULL
);

INSERT INTO monthly_revenue (month, revenue)
SELECT date_trunc('month', o.ordered_at)::date, sum(oi.quantity * oi.unit_price)
FROM orders o JOIN order_items oi USING (order_id)
WHERE o.status <> 'cancelled'
GROUP BY 1;

SELECT * FROM monthly_revenue ORDER BY month;
```

```text
CREATE TABLE
INSERT 0 6
   month    | revenue
------------+---------
 2026-01-01 | 1445.89
 2026-02-01 | 1601.98
 2026-03-01 |  436.00
 2026-04-01 | 1478.87
 2026-05-01 | 1293.00
 2026-06-01 | 1743.90
(6 rows)
```

### Data-modifying CTEs

In PostgreSQL, `INSERT`, `UPDATE`, and `DELETE` with `RETURNING` can appear inside a `WITH` clause, and their output feeds the rest of the statement. Moving cancelled orders (and their lines) out of the live tables and into an archive, atomically, in one statement:

```sql
CREATE TABLE cancelled_orders_archive (LIKE orders);

WITH removed_lines AS (
    DELETE FROM order_items
    WHERE order_id IN (SELECT order_id FROM orders WHERE status = 'cancelled')
    RETURNING order_id
),
removed_orders AS (
    DELETE FROM orders
    WHERE status = 'cancelled'
    RETURNING *
),
archived AS (
    INSERT INTO cancelled_orders_archive
    SELECT * FROM removed_orders
    RETURNING order_id
)
SELECT (SELECT count(*) FROM removed_lines) AS lines_removed,
       (SELECT count(*) FROM archived)      AS orders_archived;
```

```text
CREATE TABLE
 lines_removed | orders_archived
---------------+-----------------
             1 |               1
(1 row)
```

**Notes:**
- Always list the target columns. `INSERT INTO t VALUES (...)` without a column list breaks, or worse silently misplaces data, when the table changes.
- Insert in batches. One statement with 1,000 rows is dramatically faster than 1,000 statements, and `COPY` is faster still for bulk loads ([Section 45](#45-import-export-and-backup)).
- `RETURNING` is the clean way to get generated IDs in PostgreSQL, SQLite (3.35+), and MariaDB.
- `LIKE other_table` in `CREATE TABLE` copies column definitions; `CREATE TABLE ... AS SELECT` copies a query's result ([Section 35](#35-creating-and-altering-tables)).

---

## 33. UPDATE, DELETE, and TRUNCATE

### UPDATE

`UPDATE` changes column values in existing rows. The `WHERE` clause decides which rows — without it, **every row** is updated.

```sql
UPDATE products
SET price = price * 1.10,
    stock = stock + 5
WHERE category_id = 6
RETURNING product_id, name, price, stock;
```

```text
 product_id |          name          | price | stock
------------+------------------------+-------+-------
          8 | SQL for Everyone       | 42.90 |   205
          9 | Python Deep Dive       | 53.90 |   155
         10 | Designing Data Systems | 60.50 |    85
(3 rows)

UPDATE 3
```

All expressions on the right-hand side see the row's **old** values, so `SET a = b, b = a` swaps two columns cleanly.

### UPDATE with values from another table

`UPDATE ... FROM` joins another table (or a `VALUES` list) to supply new values. Applying a batch of stock counts from a warehouse feed:

```sql
UPDATE products AS p
SET stock = feed.stock
FROM (VALUES (3, 42), (5, 18), (11, 0)) AS feed(product_id, stock)
WHERE p.product_id = feed.product_id
RETURNING p.product_id, p.name, p.stock;
```

```text
 product_id |            name             | stock
------------+-----------------------------+-------
          3 | Mechanical Keyboard         |    42
          5 | USB-C Hub                   |    18
         11 | Noise-Cancelling Headphones |     0
(3 rows)

UPDATE 3
```

If the joined source has several matches for one target row, PostgreSQL applies an arbitrary one. Make sure the source is unique on the join key. (MySQL uses `UPDATE a JOIN b ON ... SET ...`; SQL Server uses `UPDATE a SET ... FROM a JOIN b ON ...`.)

### DELETE

`DELETE` removes rows matching `WHERE`. Foreign keys protect related data: a customer with orders cannot be deleted while those orders reference them:

```sql
DELETE FROM customers WHERE customer_id = 1;
```

```text
ERROR:  update or delete on table "customers" violates foreign key constraint "orders_customer_id_fkey" on table "orders"
DETAIL:  Key (customer_id)=(1) is still referenced from table "orders".
```

```sql
DELETE FROM customers WHERE customer_id = 10   -- Jonas has no orders
RETURNING first_name, email;
```

```text
 first_name |       email
------------+-------------------
 Jonas      | jonas@example.com
(1 row)

DELETE 1
```

`DELETE ... USING` deletes based on a join. Removing order lines for products that are out of stock and belong to pending orders:

```sql
DELETE FROM order_items AS oi
USING orders AS o
WHERE o.order_id = oi.order_id
  AND o.status = 'pending'
  AND oi.product_id = 11
RETURNING oi.order_id, oi.product_id;
```

```text
 order_id | product_id
----------+------------
       15 |         11
(1 row)

DELETE 1
```

### Deleting duplicates

When a table has exact duplicates, number the copies and delete all but the first. This uses a key that distinguishes the copies — with a real table, use its primary key or PostgreSQL's physical row id `ctid`:

```sql
CREATE TABLE signups (email text, signed_up date);
INSERT INTO signups VALUES ('a@example.com', '2026-01-01'), ('a@example.com', '2026-01-01'),
                           ('b@example.com', '2026-01-02'), ('a@example.com', '2026-01-01');

DELETE FROM signups
WHERE ctid IN (
    SELECT ctid
    FROM (SELECT ctid, row_number() OVER (PARTITION BY email, signed_up ORDER BY ctid) AS rn
          FROM signups) AS t
    WHERE rn > 1
);

SELECT * FROM signups;
```

```text
CREATE TABLE
INSERT 0 4
DELETE 2
     email     | signed_up
---------------+------------
 a@example.com | 2026-01-01
 b@example.com | 2026-01-02
(2 rows)
```

### Safe habits for UPDATE and DELETE

A forgotten `WHERE` is the classic catastrophe. Three habits prevent it:

1. **Run the `WHERE` as a `SELECT` first.** `SELECT count(*) FROM products WHERE ...` tells you how many rows the change will touch.
2. **Wrap risky changes in a transaction**, check the result, and only then commit ([Section 41](#41-transactions-isolation-and-locking)).
3. **Use `RETURNING`** to see exactly what changed.

```sql
BEGIN;
UPDATE employees SET salary = salary * 2;          -- oops: no WHERE
SELECT count(*) AS rows_doubled FROM employees WHERE salary > 150000;
ROLLBACK;                                          -- undo everything since BEGIN
SELECT max(salary) FROM employees;
```

```text
BEGIN
UPDATE 9
 rows_doubled
--------------
            6
(1 row)

ROLLBACK
    max
-----------
 180000.00
(1 row)
```

### TRUNCATE

`TRUNCATE` empties a table instantly by discarding its storage rather than deleting row by row. It is far faster than `DELETE` on large tables, but it cannot have a `WHERE`, does not fire row-level triggers, and needs `CASCADE` if other tables reference it:

```sql
TRUNCATE order_items;
TRUNCATE orders;
```

```text
TRUNCATE TABLE
ERROR:  cannot truncate a table referenced in a foreign key constraint
DETAIL:  Table "order_items" references "orders".
HINT:  Truncate table "order_items" at the same time, or use TRUNCATE ... CASCADE.
```

```sql
TRUNCATE orders CASCADE;   -- also truncates tables with foreign keys pointing here
```

```text
NOTICE:  truncate cascades to table "order_items"
TRUNCATE TABLE
```

`RESTART IDENTITY` also resets identity counters. In PostgreSQL, `TRUNCATE` is transactional and can be rolled back; in MySQL and Oracle it commits implicitly.

**Notes:**
- `UPDATE` and `DELETE` without `WHERE` affect every row. Many teams configure their SQL clients to refuse or warn about this.
- Test every destructive statement as a `SELECT` with the same `WHERE` first.
- Deleting a parent row with dependent child rows fails by default. Decide deliberately between restricting, cascading, or nulling the reference ([Section 36](#36-constraints-and-referential-integrity)).
- Many applications use **soft deletes** (a `deleted_at timestamptz` column) instead of `DELETE`, to keep history and allow undo. Every query must then filter `WHERE deleted_at IS NULL`, which a view can do for you.
- `DELETE` vs `TRUNCATE` vs `DROP` is a common interview question: `DELETE` removes chosen rows (logged, triggers fire), `TRUNCATE` removes all rows fast (keeps the table), `DROP` removes the table itself.

---

## 34. Upserts and MERGE

An **upsert** means "insert this row, or update it if it already exists". It is the core operation of syncing data, loading pipelines idempotently, and maintaining counters.

### ON CONFLICT

PostgreSQL's `INSERT ... ON CONFLICT` targets a unique constraint or primary key. `DO NOTHING` skips rows that would conflict:

```sql
INSERT INTO customers (first_name, last_name, email, country, signup_date)
VALUES ('Alice', 'Martin', 'alice@example.com', 'UK', '2025-01-05'),
       ('Mo',    'Salah',  'mo@example.com',    'Egypt', '2026-07-05')
ON CONFLICT (email) DO NOTHING
RETURNING customer_id, email;
```

```text
 customer_id |     email
-------------+----------------
          12 | mo@example.com
(1 row)

INSERT 0 1
```

Only the new row was inserted. `DO UPDATE` modifies the existing row instead; the special table `EXCLUDED` holds the values that were proposed for insertion:

```sql
CREATE TABLE product_views (
    product_id integer PRIMARY KEY REFERENCES products,
    views      integer NOT NULL,
    last_seen  timestamptz NOT NULL
);

INSERT INTO product_views VALUES (1, 1, '2026-07-01 10:00+00');

INSERT INTO product_views (product_id, views, last_seen)
VALUES (1, 1, '2026-07-01 11:00+00'),
       (2, 1, '2026-07-01 11:05+00')
ON CONFLICT (product_id) DO UPDATE
SET views     = product_views.views + EXCLUDED.views,
    last_seen = GREATEST(product_views.last_seen, EXCLUDED.last_seen);

SELECT * FROM product_views ORDER BY product_id;
```

```text
CREATE TABLE
INSERT 0 1
INSERT 0 2
 product_id | views |       last_seen
------------+-------+------------------------
          1 |     2 | 2026-07-01 11:00:00+00
          2 |     1 | 2026-07-01 11:05:00+00
(2 rows)
```

`ON CONFLICT` requires a unique index or constraint on the conflict columns; without one, the database cannot know what "already exists" means. It is also safe under concurrency: two sessions upserting the same key never create a duplicate.

### MERGE

`MERGE` (SQL standard; PostgreSQL 15+, SQL Server, Oracle, Snowflake, BigQuery, Databricks) synchronizes a target table with a source in one statement, with separate actions for matched and unmatched rows:

```sql
CREATE TABLE price_feed (product_id integer, new_price numeric(10,2), discontinued boolean);
INSERT INTO price_feed VALUES (1, 1249.00, false),   -- price change
                              (12, 59.00, true),     -- discontinued product
                              (99, 15.00, false);    -- not in products: ignored below

MERGE INTO products AS p
USING price_feed AS f
   ON p.product_id = f.product_id
WHEN MATCHED AND f.discontinued THEN
    UPDATE SET stock = 0
WHEN MATCHED THEN
    UPDATE SET price = f.new_price
WHEN NOT MATCHED THEN
    DO NOTHING;

SELECT product_id, name, price, stock FROM products WHERE product_id IN (1, 12);
```

```text
CREATE TABLE
INSERT 0 3
MERGE 2
 product_id |     name      |  price  | stock
------------+---------------+---------+-------
          1 | Laptop Pro 14 | 1249.00 |    15
         12 | Webcam HD     |   59.00 |     0
(2 rows)
```

`WHEN NOT MATCHED THEN INSERT (...) VALUES (...)` inserts new rows, and PostgreSQL 17 added `WHEN NOT MATCHED BY SOURCE` — rows in the target missing from the source, typically to delete them — and `RETURNING` for `MERGE`.

### ON CONFLICT or MERGE?

In PostgreSQL, prefer `ON CONFLICT` for simple upserts from applications — it is concise and fully concurrency-safe. Use `MERGE` for batch synchronization with several conditional branches, and in warehouses, where `MERGE` is the standard way to apply incremental loads ([Section 48](#48-data-engineering-warehouses-star-schemas-and-pipelines)). Note that PostgreSQL's `MERGE` can still raise a unique violation if another session inserts the same key concurrently.

**Notes:**
- Upserts make loads **idempotent**: running the same load twice leaves the data in the same state as running it once. This is the property that makes pipelines safely retryable.
- Identity values consumed by rows that hit a conflict are not reused. That is why the new customer above got ID 12, not 11. Gaps in generated IDs are normal and harmless — never rely on IDs being consecutive.
- `EXCLUDED` refers to the proposed row; the table name (or alias) refers to the existing row.
- Dialect spellings: MySQL `INSERT ... ON DUPLICATE KEY UPDATE`, SQLite `INSERT ... ON CONFLICT` (same as PostgreSQL) or `INSERT OR REPLACE`, SQL Server and warehouses `MERGE`.
- `INSERT OR REPLACE` (SQLite) and MySQL's `REPLACE` delete the old row and insert a new one, which resets columns you did not supply and fires delete triggers. A true upsert updates in place.

---

## 35. Creating and Altering Tables

Data Definition Language (DDL) creates and changes the structure that holds data. In production, DDL runs through versioned **migrations** — files applied in order by a tool such as Alembic, Flyway, Django migrations, or dbt — never by hand, so every environment has the same schema and every change is reviewed.

### CREATE TABLE

A table definition lists columns with types, defaults, and constraints:

```sql
CREATE TABLE reviews (
    review_id    bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    product_id   integer     NOT NULL REFERENCES products (product_id),
    customer_id  integer     NOT NULL REFERENCES customers (customer_id),
    rating       smallint    NOT NULL CHECK (rating BETWEEN 1 AND 5),
    body         text,
    created_at   timestamptz NOT NULL DEFAULT now()
);
```

```text
CREATE TABLE
```

### Identity columns and UUIDs

An **identity column** generates sequential integers. `GENERATED ALWAYS` refuses manually supplied values (the safe default); `GENERATED BY DEFAULT` allows them, which is convenient when loading data that already has IDs (the sample database uses this form). Older code uses `serial` / `bigserial`, which does the same job less cleanly; MySQL uses `AUTO_INCREMENT` and SQL Server `IDENTITY(1,1)`.

```sql
INSERT INTO reviews (review_id, product_id, customer_id, rating) VALUES (100, 1, 1, 5);
```

```text
ERROR:  cannot insert a non-DEFAULT value into column "review_id"
DETAIL:  Column "review_id" is an identity column defined as GENERATED ALWAYS.
HINT:  Use OVERRIDING SYSTEM VALUE to override.
```

**UUIDs** are 128-bit identifiers that can be generated anywhere — in the application, offline, across databases — without coordination, and they do not reveal how many rows exist. `gen_random_uuid()` generates random (version 4) UUIDs; PostgreSQL 18 adds `uuidv7()`, whose time-ordered values index much better:

```sql
SELECT gen_random_uuid() IS NOT NULL AS works, pg_typeof(gen_random_uuid()) AS type;
```

```text
 works | type
-------+------
 t     | uuid
(1 row)
```

### Creating tables from queries

`CREATE TABLE ... AS SELECT` (CTAS) creates and fills a table from a query in one step. It copies data and column types but no constraints, indexes, or defaults. It is used heavily for snapshots and analysis tables:

```sql
CREATE TABLE customer_summary AS
SELECT c.customer_id, c.country, count(o.order_id) AS orders
FROM customers c LEFT JOIN orders o USING (customer_id)
GROUP BY c.customer_id, c.country;

SELECT * FROM customer_summary ORDER BY orders DESC LIMIT 3;
```

```text
SELECT 10
 customer_id | country | orders
-------------+---------+--------
           1 | UK      |      3
           3 | Canada  |      3
           5 | Canada  |      2
(3 rows)
```

**Temporary tables** (`CREATE TEMP TABLE ...`) exist only for the current session and vanish on disconnect, which makes them useful scratch space for multi-step scripts.

### ALTER TABLE

`ALTER TABLE` changes an existing table:

```sql
ALTER TABLE customers ADD COLUMN phone text;
ALTER TABLE customers ADD COLUMN is_vip boolean NOT NULL DEFAULT false;
ALTER TABLE customers RENAME COLUMN phone TO phone_number;
ALTER TABLE customers ALTER COLUMN city SET DEFAULT 'Unknown';
ALTER TABLE customers DROP COLUMN phone_number;
ALTER TABLE products  ALTER COLUMN stock TYPE bigint;
```

```text
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
```

```sql
SELECT customer_id, first_name, is_vip FROM customers LIMIT 2;
```

```text
 customer_id | first_name | is_vip
-------------+------------+--------
           1 | Alice      | f
           2 | Bruno      | f
(2 rows)
```

### Generated columns

A **generated column** is computed from other columns and kept up to date automatically. It cannot be written directly:

```sql
ALTER TABLE order_items
ADD COLUMN line_total numeric(12,2) GENERATED ALWAYS AS (quantity * unit_price) STORED;

SELECT order_id, product_id, quantity, unit_price, line_total FROM order_items WHERE order_id = 4;
```

```text
ALTER TABLE
 order_id | product_id | quantity | unit_price | line_total
----------+------------+----------+------------+------------
        4 |          2 |        1 |     999.00 |     999.00
        4 |          5 |        1 |      45.00 |      45.00
        4 |          4 |        2 |      29.99 |      59.98
(3 rows)
```

### Schemas

A **schema** is a namespace inside a database: a folder for tables. The default is `public`. Separate schemas are useful for grouping (`staging`, `analytics`, `audit`) and for permissions. Objects are addressed as `schema.table`:

```sql
CREATE SCHEMA analytics;
CREATE TABLE analytics.daily_sales (day date PRIMARY KEY, revenue numeric(12,2));
SELECT table_schema, table_name FROM information_schema.tables
WHERE table_schema IN ('public', 'analytics') ORDER BY 1, 2;
```

```text
CREATE SCHEMA
CREATE TABLE
 table_schema |    table_name
--------------+------------------
 analytics    | daily_sales
 public       | categories
 public       | customer_summary
 public       | customers
 public       | employees
 public       | order_items
 public       | orders
 public       | products
 public       | reviews
(9 rows)
```

`information_schema` (standard) and `pg_catalog` (PostgreSQL-specific) are built-in schemas that describe the database itself. Querying them is how tools, and you, discover tables, columns, and constraints programmatically.

### DROP

`DROP TABLE` deletes the table and all its data. `IF EXISTS` avoids an error when it is absent, and `CASCADE` also drops dependent objects such as views and foreign keys pointing at it:

```sql
DROP TABLE IF EXISTS customer_summary;
DROP TABLE IF EXISTS no_such_table;
```

```text
DROP TABLE
NOTICE:  table "no_such_table" does not exist, skipping
DROP TABLE
```

**Notes:**
- Adding a nullable column, or (since PostgreSQL 11) a column with a constant default, is instant even on huge tables. Changing a column's type usually rewrites the whole table under an exclusive lock, so schedule it carefully.
- `DROP` and `TRUNCATE` are not undoable outside a transaction. In PostgreSQL, DDL *is* transactional (you can `BEGIN; DROP TABLE x; ROLLBACK;`), which not every database offers: MySQL and Oracle commit DDL implicitly.
- Use `bigint` identity columns for tables that may grow large, and UUIDs when IDs are generated outside the database or must not be guessable.
- Keep all schema changes in migration files under version control. A schema changed by hand in one environment is a bug waiting to surface in another.

---

## 36. Constraints and Referential Integrity

**Constraints** are rules the database enforces on every write, forever, no matter which application, script, or person writes the data. They are the cheapest and most reliable data-quality tool you have, because bad data never gets in.

| Constraint | Guarantees |
|:--|:--|
| `NOT NULL` | A value is always present |
| `UNIQUE` | No two rows share the value (or combination of values) |
| `PRIMARY KEY` | `UNIQUE` + `NOT NULL`, the row's identity |
| `FOREIGN KEY` (`REFERENCES`) | The value exists in the referenced table |
| `CHECK` | A boolean condition holds for every row |
| `DEFAULT` | Not a constraint strictly, but supplies a value when none is given |
| `EXCLUDE` | No two rows "overlap" by a given operator (PostgreSQL; for example, no double-booked time ranges) |

### CHECK constraints

A `CHECK` can express any rule about one row, including rules across several columns:

```sql
CREATE TABLE promotions (
    promo_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    code        text NOT NULL UNIQUE CHECK (code = upper(code)),
    discount    numeric(4,2) NOT NULL CHECK (discount > 0 AND discount <= 0.5),
    starts_on   date NOT NULL,
    ends_on     date NOT NULL,
    CONSTRAINT valid_period CHECK (ends_on > starts_on)
);

INSERT INTO promotions (code, discount, starts_on, ends_on)
VALUES ('SUMMER', 0.2, '2026-07-01', '2026-06-01');
```

```text
CREATE TABLE
ERROR:  new row for relation "promotions" violates check constraint "valid_period"
DETAIL:  Failing row contains (1, SUMMER, 0.20, 2026-07-01, 2026-06-01).
```

Naming constraints (`CONSTRAINT valid_period ...`) makes error messages self-explanatory. Unnamed ones get generated names like `promotions_discount_check`.

### Multi-column UNIQUE

`UNIQUE (a, b)` allows repeated values in `a` or `b` alone but not the same pair twice. One review per customer per product:

```sql
CREATE TABLE ratings (
    customer_id integer REFERENCES customers,
    product_id  integer REFERENCES products,
    stars       smallint NOT NULL,
    UNIQUE (customer_id, product_id)
);
INSERT INTO ratings VALUES (1, 1, 5), (1, 2, 4), (2, 1, 3);
INSERT INTO ratings VALUES (1, 1, 1);
```

```text
CREATE TABLE
INSERT 0 3
ERROR:  duplicate key value violates unique constraint "ratings_customer_id_product_id_key"
DETAIL:  Key (customer_id, product_id)=(1, 1) already exists.
```

### Foreign key actions

When a referenced row is deleted or its key updated, the foreign key's `ON DELETE` / `ON UPDATE` action decides what happens to referencing rows:

| Action | Effect on child rows |
|:--|:--|
| `NO ACTION` / `RESTRICT` (default) | The parent change is rejected while children exist |
| `CASCADE` | Children are deleted (or updated) too |
| `SET NULL` | The child's foreign key becomes `NULL` |
| `SET DEFAULT` | The child's foreign key becomes its default |

```sql
CREATE TABLE wishlists (
    wishlist_id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id integer NOT NULL REFERENCES customers ON DELETE CASCADE
);
CREATE TABLE wishlist_items (
    wishlist_id integer REFERENCES wishlists ON DELETE CASCADE,
    product_id  integer REFERENCES products,
    PRIMARY KEY (wishlist_id, product_id)
);
INSERT INTO wishlists (customer_id) VALUES (10);
INSERT INTO wishlist_items VALUES (1, 7), (1, 12);

DELETE FROM customers WHERE customer_id = 10;   -- cascades to wishlists, then wishlist_items
SELECT count(*) AS remaining_items FROM wishlist_items;
```

```text
CREATE TABLE
CREATE TABLE
INSERT 0 1
INSERT 0 2
DELETE 1
 remaining_items
-----------------
               0
(1 row)
```

Choose per relationship: `CASCADE` for things that cannot exist without their parent (a wishlist without a customer), `RESTRICT` for records that must be preserved (orders when someone tries to delete a customer), `SET NULL` for optional links (a sales rep who leaves).

### Adding constraints to existing tables

`ALTER TABLE ... ADD CONSTRAINT` validates every existing row, which fails if old data breaks the rule:

```sql
ALTER TABLE customers ADD CONSTRAINT city_required CHECK (city IS NOT NULL);
```

```text
ERROR:  check constraint "city_required" of relation "customers" is violated by some row
```

`NOT VALID` adds the constraint for new writes only, so you can fix old rows later and then run `ALTER TABLE ... VALIDATE CONSTRAINT`. On large tables this also avoids a long lock.

**Notes:**
- Push every rule you can into constraints. Application-level validation is bypassed by scripts, migrations, admin tools, and bugs — constraints are not.
- A foreign key column should almost always be indexed, otherwise deleting a parent row scans the entire child table ([Section 39](#39-indexes)).
- `UNIQUE` allows multiple `NULL`s by default. PostgreSQL 15 added `UNIQUE NULLS NOT DISTINCT` to treat `NULL`s as equal.
- Constraints can be `DEFERRABLE INITIALLY DEFERRED`, which checks them at commit instead of per statement. This is useful for circular references and bulk reorganizations.
- MySQL ignored `CHECK` constraints entirely before version 8.0.16. Warehouses (Snowflake, BigQuery, Redshift) mostly do not enforce `PRIMARY KEY`, `UNIQUE`, or `FOREIGN KEY`. Know which guarantees your engine really gives.

---

## 37. Normalization and Schema Design

**Normalization** is the process of organizing tables so that each fact is stored exactly once. It prevents *anomalies*: data that contradicts itself because the same fact lives in several places.

### The problem with one big table

Imagine storing orders in a single flat table:

```text
order_id | customer_name | customer_email    | product        | category    | price  | quantity
---------+---------------+-------------------+----------------+-------------+--------+---------
       1 | Alice Martin  | alice@example.com | Laptop Pro 14  | Laptops     | 1199.00|        1
       1 | Alice Martin  | alice@example.com | Wireless Mouse | Accessories |  29.99 |        1
       3 | Alice Martin  | alice@example.com | Keyboard       | Accessories |  89.90 |        1
```

- **Update anomaly:** Alice changes her email — it must change on every row, and missing one leaves two emails for one person.
- **Insert anomaly:** a new product cannot be recorded until someone orders it.
- **Delete anomaly:** deleting the only order for a product erases the fact that the product exists.

### The normal forms

Each normal form removes one kind of redundancy. The first three are the ones that matter in practice:

| Form | Rule | Violation | Fix |
|:--|:--|:--|:--|
| **1NF** | Each cell holds one atomic value; no repeating groups | A `products` column holding `"Laptop, Mouse"` | One row per item, in a child table |
| **2NF** | 1NF, and every non-key column depends on the **whole** key | In `order_items (order_id, product_id, product_name)`, `product_name` depends only on `product_id` | Move `product_name` to `products` |
| **3NF** | 2NF, and non-key columns depend **only on the key**, not on other non-key columns | `customers (customer_id, city, country_of_city)` where country is determined by city | Move the dependency to its own table, or accept it knowingly |

The summary every database course quotes — each non-key column must depend on "the key, the whole key, and nothing but the key". **BCNF** tightens 3NF for tables with overlapping candidate keys; 4NF and 5NF address multi-valued dependencies and are rarely a practical concern.

The sample database is in 3NF: customers, products, and categories each live in their own table, orders reference customers, and `order_items` resolves the many-to-many relationship.

### Facts that look like duplication but are not

`order_items.unit_price` looks like it duplicates `products.price`. It does not: it records a *different fact*, the price paid at the moment of sale, which must not change when the catalog price changes. The same reasoning applies to shipping addresses on orders and exchange rates on transactions. The test is not "do two columns hold the same value today?" but "do they represent the same fact forever?"

### Denormalization

Normalized schemas are ideal for **writing** (OLTP). For **reading** large volumes (OLAP), joins across many tables get expensive, so analytics schemas deliberately denormalize — they copy descriptive attributes into wide tables to make queries simple and fast. The star schema in [Section 48](#48-data-engineering-warehouses-star-schemas-and-pipelines) is a structured form of this. Denormalize on purpose, for measured read performance, and preferably in derived tables or materialized views rather than in the source of truth.

### Practical design checklist

- Every table has a primary key, preferably a surrogate `bigint` identity or UUID.
- Natural identifiers (email, SKU, ISBN) get `UNIQUE` constraints.
- Columns are `NOT NULL` unless absence is genuinely meaningful.
- Foreign keys are declared and indexed.
- Money is `numeric`, times are `timestamptz`, and flags are `boolean`.
- Names are consistent: plural or singular tables (pick one), `snake_case`, keys named `<table>_id`.
- Every table has `created_at` (and often `updated_at`) timestamps — you will want them for debugging and incremental loads.
- Fixed small sets of values use a `CHECK (x IN (...))`, an enum type, or a lookup table with a foreign key — the lookup table is easiest to extend.

### An anti-pattern: entity-attribute-value

The **EAV** design stores everything in one generic `(entity_id, attribute, value)` table so that "any attribute can be added without a migration". It destroys typing, constraints, and query simplicity (every read needs a pivot). When records genuinely have varying attributes, a `jsonb` column ([Section 40](#40-json-arrays-and-full-text-search)) alongside normal columns is almost always better, which is what the sample database does with `products.attributes`.

**Notes:**
- "Explain normalization and the normal forms" is a standard interview question. Give the anomaly motivation first, then 1NF to 3NF with one example each.
- Normalize the source of truth (OLTP), denormalize the reporting layer (OLAP). This one sentence resolves most normalization debates.
- Storing derived values (a customer's order count) creates a second copy that can drift. Compute it, or maintain it with a trigger or materialized view, and document which is the source of truth.
- Comma-separated values in a column violate 1NF and make every query harder. Use a child table or an array/JSON column with a clear reason.

---

## 38. Views and Materialized Views

### Views

A **view** is a saved query that behaves like a table. It stores no data — every time you query it, its query runs. Views give complex logic a name, hide complexity from consumers, and control what data is exposed:

```sql
CREATE VIEW order_summary AS
SELECT o.order_id,
       o.ordered_at::date                    AS order_date,
       c.first_name || ' ' || c.last_name    AS customer,
       o.status,
       count(*)                              AS lines,
       sum(oi.quantity * oi.unit_price)      AS total
FROM orders o
JOIN customers c   USING (customer_id)
JOIN order_items oi USING (order_id)
GROUP BY o.order_id, c.first_name, c.last_name;

SELECT * FROM order_summary WHERE total > 500 ORDER BY total DESC;
```

```text
CREATE VIEW
 order_id | order_date |   customer    |  status   | lines |  total
----------+------------+---------------+-----------+-------+---------
       15 | 2026-06-20 | Chloe Kim     | pending   |     2 | 1498.00
       10 | 2026-04-11 | Grace Liu     | delivered |     2 | 1388.90
        1 | 2026-01-05 | Alice Martin  | delivered |     2 | 1228.99
        4 | 2026-02-03 | Chloe Kim     | delivered |     3 | 1103.98
       13 | 2026-05-19 | Daniel Okafor | shipped   |     1 |  999.00
(5 rows)
```

Queries against a view are merged with the view's definition before planning, so filters on the view can still use indexes on the underlying tables. `CREATE OR REPLACE VIEW` changes the definition (only by adding columns at the end, in PostgreSQL); `DROP VIEW` removes it.

Simple views (one table, no aggregates) are **updatable**: `INSERT`, `UPDATE`, and `DELETE` through them modify the base table. `WITH CHECK OPTION` prevents writes through the view that the view itself would not show.

### Views for security and simplicity

A view can expose a subset of columns and rows. Grant access to the view, not the table, and the underlying data stays protected ([Section 44](#44-roles-permissions-and-row-level-security)):

```sql
CREATE VIEW public_customers AS
SELECT customer_id, first_name, country      -- no email, no surname
FROM customers;

SELECT * FROM public_customers LIMIT 3;
```

```text
CREATE VIEW
 customer_id | first_name | country
-------------+------------+----------
           1 | Alice      | UK
           2 | Bruno      | Portugal
           3 | Chloe      | Canada
(3 rows)
```

### Materialized views

A **materialized view** stores the result of its query physically, like a table. Reading it is as fast as reading a table, but its data is a snapshot that goes stale until you refresh it:

```sql
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT date_trunc('month', o.ordered_at)::date AS month,
       sum(oi.quantity * oi.unit_price)       AS revenue
FROM orders o JOIN order_items oi USING (order_id)
WHERE o.status <> 'cancelled'
GROUP BY 1;

INSERT INTO orders (customer_id, ordered_at, status) VALUES (9, '2026-06-25', 'delivered');
INSERT INTO order_items VALUES (currval('orders_order_id_seq'), 7, 1, 1499.00);

SELECT * FROM monthly_sales WHERE month = '2026-06-01';   -- stale: still the old total
REFRESH MATERIALIZED VIEW monthly_sales;
SELECT * FROM monthly_sales WHERE month = '2026-06-01';   -- refreshed
```

```text
SELECT 6
INSERT 0 1
INSERT 0 1
   month    | revenue
------------+---------
 2026-06-01 | 1743.90
(1 row)

REFRESH MATERIALIZED VIEW
   month    | revenue
------------+---------
 2026-06-01 | 3242.90
(1 row)
```

`REFRESH ... CONCURRENTLY` rebuilds without blocking readers, but requires a unique index on the materialized view. Refreshes are typically scheduled (every hour, every night) by a job runner.

| | View | Materialized view | Table |
|:--|:--|:--|:--|
| Stores data | No | Yes (a snapshot) | Yes |
| Always current | Yes | Only after refresh | You maintain it |
| Read speed | Same as the query | Fast | Fast |
| Typical use | Naming logic, security | Expensive dashboards and aggregates | Source of truth |

**Notes:**
- A view is a stored query, not stored data. It is only as fast as the query behind it.
- Views stacked on views on views become hard to debug and optimize. Keep the layers shallow.
- Materialized views are a controlled form of denormalization — fast reads, bounded staleness, one refresh command to keep them honest.
- MySQL has no materialized views; SQL Server has "indexed views" that update automatically; warehouses (Snowflake, BigQuery) offer materialized views with automatic refresh. In dbt, materialization is a per-model setting (`view`, `table`, `incremental`).

---

## 39. Indexes

An **index** is a separate data structure that lets the database find rows without reading the whole table, exactly like the index at the back of a book. Indexes are the most important performance tool in SQL, and choosing them well is a core professional skill.

### How a B-tree index works

The default index type is a **B-tree**: a balanced, sorted tree of the indexed values, each pointing to the row's location. Finding a value takes a few page reads regardless of table size — a billion rows is only about four or five levels deep — and because the values are sorted, the same index serves equality (`=`), ranges (`<`, `BETWEEN`), prefix `LIKE 'abc%'`, `ORDER BY`, and `min`/`max`.

```text
                  [ 400 | 800 ]
                 /      |      \
     [100|200|300]  [500|600|700]  [900|...]
       /  |  |  \      ...
    leaf pages: sorted values → row locations
```

Without a suitable index, the database performs a **sequential scan**: it reads every row and tests the condition. That is the right choice for small tables and for queries that return a large fraction of rows, and a disaster for finding a few rows in a large table.

### Seeing the difference

The sample tables are too small for indexes to matter, so this section builds a million-row table:

```sql
CREATE TABLE events (
    event_id   bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id    integer     NOT NULL,
    event_type text        NOT NULL,
    created_at timestamptz NOT NULL
);

SELECT setseed(0.42);          -- make random() repeatable

INSERT INTO events (user_id, event_type, created_at)
SELECT (random() * 50000)::int,
       (ARRAY['view', 'click', 'purchase'])[1 + (random() * 2)::int],
       '2026-01-01'::timestamptz + random() * interval '180 days'
FROM generate_series(1, 1000000);

ANALYZE events;
```

```text
CREATE TABLE
 setseed
---------

(1 row)

INSERT 0 1000000
ANALYZE
```

`EXPLAIN` shows the plan the database chose ([Section 42](#42-query-plans-and-performance) covers reading plans in depth). Finding one user's events without an index:

```sql
EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT count(*) FROM events WHERE user_id = 4242;
```

```text
                              QUERY PLAN
-----------------------------------------------------------------------
 Finalize Aggregate (actual rows=1 loops=1)
   ->  Gather (actual rows=3 loops=1)
         Workers Planned: 2
         Workers Launched: 2
         ->  Partial Aggregate (actual rows=1 loops=3)
               ->  Parallel Seq Scan on events (actual rows=7 loops=3)
                     Filter: (user_id = 4242)
                     Rows Removed by Filter: 333326
 Planning Time: 0.254 ms
 Execution Time: 56.430 ms
(10 rows)
```

The plan reads the whole table (a parallel sequential scan) and throws away almost every row. Now add an index and run the same query:

```sql
CREATE INDEX idx_events_user_id ON events (user_id);

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT count(*) FROM events WHERE user_id = 4242;
```

```text
CREATE INDEX
                                  QUERY PLAN
------------------------------------------------------------------------------
 Aggregate (actual rows=1 loops=1)
   ->  Bitmap Heap Scan on events (actual rows=21 loops=1)
         Recheck Cond: (user_id = 4242)
         Heap Blocks: exact=21
         ->  Bitmap Index Scan on idx_events_user_id (actual rows=21 loops=1)
               Index Cond: (user_id = 4242)
 Planning Time: 0.321 ms
 Execution Time: 0.137 ms
(8 rows)
```

The planner now walks the index to find the 21 matching rows (a *bitmap index scan*), then fetches just those rows from the table (the *bitmap heap scan*), instead of filtering a million. Execution time drops by more than two orders of magnitude, and the gap widens as the table grows.

### Composite indexes and column order

An index on several columns is sorted by the first column, then the second within ties of the first, like a phone book sorted by last name then first name. It can serve queries that filter on a **leftmost prefix** of its columns:

| Index on `(user_id, created_at)` | Can use the index? |
|:--|:--|
| `WHERE user_id = 7` | Yes (leftmost column) |
| `WHERE user_id = 7 AND created_at >= '2026-03-01'` | Yes, both columns, ideal |
| `WHERE user_id = 7 ORDER BY created_at DESC LIMIT 10` | Yes, and it avoids the sort |
| `WHERE created_at >= '2026-03-01'` | Not efficiently (skips the leading column) |

The rule of thumb: put columns tested with **equality first**, then the column used for **ranges or sorting**.

### Specialized indexes

```sql
-- Partial index: only index the rows that queries actually look for
CREATE INDEX idx_orders_pending ON orders (ordered_at) WHERE status = 'pending';

-- Expression index: index the result of a function, for case-insensitive lookups
CREATE INDEX idx_customers_email_lower ON customers (lower(email));

-- Unique index: enforces uniqueness, like a UNIQUE constraint
CREATE UNIQUE INDEX idx_products_name ON products (name);

-- Covering index: INCLUDE extra columns so queries never visit the table
CREATE INDEX idx_events_user_time ON events (user_id, created_at) INCLUDE (event_type);
```

```text
CREATE INDEX
CREATE INDEX
CREATE INDEX
CREATE INDEX
```

An expression index is used only when the query uses the same expression: `WHERE lower(email) = 'alice@example.com'` can use `idx_customers_email_lower`, but `WHERE email = ...` cannot.

| Index type | Good for |
|:--|:--|
| **B-tree** (default) | Equality, ranges, sorting, prefix `LIKE`; almost everything |
| **Hash** | Equality only; rarely better than B-tree |
| **GIN** | "Contains" queries: `jsonb`, arrays, full-text search ([Section 40](#40-json-arrays-and-full-text-search)) |
| **GiST** / **SP-GiST** | Geometric and range data, nearest-neighbor, exclusion constraints |
| **BRIN** | Huge, naturally ordered tables (append-only time series); tiny and cheap |
| **HNSW** / **IVFFlat** (pgvector) | Approximate nearest neighbor on embeddings ([Section 50](#50-sql-for-machine-learning-and-ai-features-splits-and-vector-search)) |

### When an index does not help

Indexes are not free, and they are not always used:

- **Every index slows down writes**: each `INSERT`, `UPDATE`, and `DELETE` must update every index on the table. Indexes also take disk space and memory.
- **Low selectivity**: if a condition matches a large share of rows (`status = 'delivered'` on 70 percent of orders), a sequential scan is cheaper, and the planner will rightly ignore the index.
- **Non-sargable predicates**: wrapping the indexed column in a function or calculation (`WHERE created_at::date = '2026-03-01'`, `WHERE price * 1.24 > 100`, `WHERE lower(email) = ...` without an expression index) hides it from the index. Rewrite the predicate so the bare column is compared: `WHERE created_at >= '2026-03-01' AND created_at < '2026-03-02'`.
- **Leading wildcards**: `LIKE '%term'` cannot use a B-tree (a trigram GIN index can).
- **Type mismatches**: comparing an indexed `integer` column to a `text` value forces a conversion on the column.

**Notes:**
- Index the columns you filter, join, and sort on in frequent queries, and index every foreign key. Do not index everything — measure.
- Primary keys and `UNIQUE` constraints create indexes automatically. Foreign keys do not.
- A composite index `(a, b)` makes a separate index on `(a)` redundant, but not one on `(b)`.
- `CREATE INDEX CONCURRENTLY` builds an index without blocking writes, which is essential on busy production tables.
- "Why is my query not using the index?" has four usual answers — the predicate is not sargable, the condition is not selective enough, the statistics are stale (`ANALYZE`), or the column order of a composite index does not match the query.

---

## 40. JSON, Arrays, and Full-Text Search

PostgreSQL goes beyond flat relational columns, which lets one database cover workloads that would otherwise need a document store or a search engine.

### jsonb

`jsonb` stores JSON in a parsed binary form that can be indexed and queried efficiently (plain `json` stores the text as-is and is rarely the right choice). The key operators:

| Operator | Returns | Example |
|:--|:--|:--|
| `->` | JSON value by key or index | `attributes -> 'brand'` gives `"Nova"` (still JSON) |
| `->>` | Text value by key or index | `attributes ->> 'brand'` gives `Nova` |
| `#>>` | Text value at a path | `data #>> '{address,city}'` |
| `@>` | Does the left contain the right? | `attributes @> '{"brand": "Nova"}'` |
| `?` | Does the key exist? | `attributes ? 'ram_gb'` |

```sql
SELECT name,
       attributes ->> 'brand'               AS brand,
       (attributes ->> 'ram_gb')::int       AS ram_gb,
       attributes ? 'color'                 AS has_color
FROM products
WHERE attributes @> '{"brand": "Nova"}'
ORDER BY ram_gb;
```

```text
     name      | brand | ram_gb | has_color
---------------+-------+--------+-----------
 Laptop Air 13 | Nova  |      8 | t
 Laptop Pro 14 | Nova  |     16 | t
 Desktop Tower | Nova  |     32 | f
(3 rows)
```

`->>` always returns text, so cast before comparing numbers (`(attributes ->> 'ram_gb')::int > 8`), otherwise `'16' < '8'` compares as text.

### Building and modifying JSON

SQL results can be turned into JSON directly, which is how many APIs serve nested responses straight from the database:

```sql
SELECT jsonb_build_object(
           'customer', c.first_name,
           'orders',   jsonb_agg(jsonb_build_object('id', o.order_id, 'status', o.status)
                                 ORDER BY o.order_id)
       ) AS doc
FROM customers c
JOIN orders o USING (customer_id)
WHERE c.customer_id = 1
GROUP BY c.first_name;
```

```text
                                                                   doc
-----------------------------------------------------------------------------------------------------------------------------------------
 {"orders": [{"id": 1, "status": "delivered"}, {"id": 3, "status": "delivered"}, {"id": 9, "status": "delivered"}], "customer": "Alice"}
(1 row)
```

```sql
UPDATE products
SET attributes = jsonb_set(attributes, '{warranty_years}', '2') || '{"refurbished": false}'
WHERE product_id = 1
RETURNING attributes;
```

```text
                                          attributes
-----------------------------------------------------------------------------------------------
 {"brand": "Nova", "color": "silver", "ram_gb": 16, "refurbished": false, "warranty_years": 2}
(1 row)

UPDATE 1
```

`jsonb_set` sets a path, `||` merges objects, and `-` removes a key (`attributes - 'color'`).

### Expanding JSON into rows

Semi-structured input (API payloads, event logs) often needs to become relational rows. `jsonb_array_elements` expands an array; `jsonb_to_recordset` maps objects to typed columns:

```sql
SELECT *
FROM jsonb_to_recordset('[{"sku": "A1", "qty": 2, "price": 9.5},
                          {"sku": "B7", "qty": 1, "price": 20}]')
     AS item(sku text, qty int, price numeric);
```

```text
 sku | qty | price
-----+-----+-------
 A1  |   2 |   9.5
 B7  |   1 |    20
(2 rows)
```

A **GIN index** makes containment and key-existence queries fast on large tables: `CREATE INDEX ON products USING gin (attributes);`.

### Arrays

PostgreSQL columns can hold arrays of any type. They suit small lists that are always read together with their row (tags, labels):

```sql
SELECT ARRAY['sql', 'data', 'python']            AS tags,
       'data' = ANY (ARRAY['sql', 'data'])        AS has_data,
       ARRAY[1, 2, 3] @> ARRAY[2]                 AS contains,
       array_length(ARRAY[1, 2, 3], 1)            AS len,
       (ARRAY['a', 'b', 'c'])[2]                  AS second;       -- arrays are 1-based
```

```text
       tags        | has_data | contains | len | second
-------------------+----------+----------+-----+--------
 {sql,data,python} | t        | t        |   3 | b
(1 row)
```

`unnest` turns an array into rows, the inverse of `array_agg`:

```sql
SELECT order_id, unnest(ARRAY['gift-wrap', 'express']) AS option
FROM orders WHERE order_id = 1;
```

```text
 order_id |  option
----------+-----------
        1 | gift-wrap
        1 | express
(2 rows)
```

### Full-text search

`LIKE '%word%'` is slow on large text and understands nothing about language. Full-text search converts text into a `tsvector` (normalized word stems) and queries into a `tsquery`, matched with `@@`:

```sql
SELECT to_tsvector('english', 'Designing data-intensive systems for engineers') AS vector;
```

```text
                               vector
---------------------------------------------------------------------
 'data':3 'data-intens':2 'design':1 'engin':7 'intens':4 'system':5
(1 row)
```

Words are lowercased and stemmed ("engineers" becomes "engin"), and stop words are dropped. Searching and ranking products by name:

```sql
SELECT name,
       ts_rank(to_tsvector('english', name), query) AS rank
FROM products,
     websearch_to_tsquery('english', 'laptop OR systems') AS query
WHERE to_tsvector('english', name) @@ query
ORDER BY rank DESC, name;
```

```text
          name          |    rank
------------------------+-------------
 Designing Data Systems | 0.030396355
 Laptop Air 13          | 0.030396355
 Laptop Pro 14          | 0.030396355
(3 rows)
```

`websearch_to_tsquery` accepts search-box syntax (quotes, `OR`, `-exclude`). In production, store the vector in a generated column with a GIN index so searches do not recompute it. For fuzzy matching of typos, the `pg_trgm` extension provides trigram similarity and makes `LIKE '%term%'` indexable.

**Notes:**
- Use `jsonb` for genuinely variable attributes, not to avoid designing a schema. Columns you filter, join, or aggregate on regularly deserve real typed columns.
- `->` returns JSON and `->>` returns text. Most bugs with JSON in SQL are a wrong arrow or a missing cast.
- JSON syntax varies widely: MySQL uses `JSON_EXTRACT(col, '$.brand')` or `col->>'$.brand'`, SQL Server `JSON_VALUE`, BigQuery `JSON_VALUE`, Snowflake `col:brand`. PostgreSQL 17 added the standard `JSON_TABLE` and `JSON_VALUE` functions too.
- Arrays are convenient for small, self-contained lists. If you need to join on the elements or constrain them with foreign keys, use a child table.
- PostgreSQL full-text search covers most application search needs. Dedicated engines (Elasticsearch, OpenSearch) make sense at very large scale or for advanced relevance tuning.

---

## 41. Transactions, Isolation, and Locking

A **transaction** groups statements into one all-or-nothing unit. Moving money, placing an order and reducing stock, or archiving and deleting rows must either happen completely or not at all. Transactions are what make that guarantee possible.

### ACID

Transactions provide four properties, known by the acronym **ACID**:

| Property | Meaning | What it protects against |
|:--|:--|:--|
| **Atomicity** | All statements in the transaction take effect, or none do | Half-finished changes after an error or crash |
| **Consistency** | Every transaction moves the database from one valid state to another; all constraints hold | Violated rules, orphaned rows |
| **Isolation** | Concurrent transactions do not see each other's in-progress work (to a configurable degree) | Reading half-written data, interference between users |
| **Durability** | Once committed, changes survive crashes and power loss | Lost data after a restart |

### BEGIN, COMMIT, ROLLBACK

By default, every statement runs in its own transaction and commits immediately (**autocommit**). `BEGIN` starts an explicit transaction; `COMMIT` makes its changes permanent and visible to others; `ROLLBACK` discards them. Placing an order and reserving stock together:

```sql
BEGIN;
INSERT INTO orders (customer_id, employee_id, ordered_at, status)
VALUES (9, 7, '2026-07-10 12:00+00', 'pending');
INSERT INTO order_items VALUES (currval('orders_order_id_seq'), 2, 1, 999.00);
UPDATE products SET stock = stock - 1 WHERE product_id = 2;
COMMIT;

SELECT o.order_id, o.status, p.stock
FROM orders o JOIN order_items oi USING (order_id) JOIN products p USING (product_id)
WHERE o.customer_id = 9;
```

```text
BEGIN
INSERT 0 1
INSERT 0 1
UPDATE 1
COMMIT
 order_id | status  | stock
----------+---------+-------
       16 | pending |    19
(1 row)
```

### Errors abort the transaction

In PostgreSQL, once any statement in a transaction fails, the whole transaction is marked as failed and every further statement is refused until you `ROLLBACK`. Nothing from it can be committed:

```sql
BEGIN;
UPDATE products SET stock = stock + 100 WHERE product_id = 3;
INSERT INTO products (name, category_id, price) VALUES ('Broken', 4, -5);   -- violates CHECK
SELECT stock FROM products WHERE product_id = 3;
ROLLBACK;
SELECT stock FROM products WHERE product_id = 3;
```

```text
BEGIN
UPDATE 1
ERROR:  new row for relation "products" violates check constraint "products_price_check"
DETAIL:  Failing row contains (13, Broken, 4, -5.00, 0, null).
ERROR:  current transaction is aborted, commands ignored until end of transaction block
ROLLBACK
 stock
-------
    50
(1 row)
```

The stock update was discarded along with the failed insert — atomicity at work.

### Savepoints

A **savepoint** marks a point inside a transaction that you can roll back to without abandoning the whole transaction. Drivers and ORMs use savepoints to implement nested transactions:

```sql
BEGIN;
UPDATE products SET price = 30.99 WHERE product_id = 4;
SAVEPOINT before_risky_change;
UPDATE products SET price = 0.01 WHERE product_id = 4;          -- a mistake
ROLLBACK TO SAVEPOINT before_risky_change;
COMMIT;
SELECT price FROM products WHERE product_id = 4;
```

```text
BEGIN
UPDATE 1
SAVEPOINT
UPDATE 1
ROLLBACK
COMMIT
 price
-------
 30.99
(1 row)
```

### Isolation levels and the anomalies they prevent

When transactions run concurrently, several anomalies are possible. The SQL standard defines four isolation levels by which anomalies they allow:

| Anomaly | What happens |
|:--|:--|
| **Dirty read** | Reading another transaction's uncommitted changes |
| **Non-repeatable read** | Reading the same row twice in one transaction and getting different values, because another transaction committed in between |
| **Phantom read** | Re-running a query and getting new rows that another transaction inserted |
| **Lost update** | Two transactions read a value, both compute a new value, and the second write overwrites the first |
| **Write skew** | Two transactions each check a condition, then write, together breaking a rule neither broke alone |

| Level | PostgreSQL behavior |
|:--|:--|
| Read Uncommitted | Treated as Read Committed (PostgreSQL never allows dirty reads) |
| **Read Committed** (default) | Each *statement* sees data committed before it started |
| Repeatable Read | The whole *transaction* sees one snapshot from its first statement; concurrent updates to the same row cause a serialization error |
| Serializable | Transactions behave as if run one at a time; conflicts cause a serialization error that the application must retry |

PostgreSQL implements isolation with **MVCC** (multi-version concurrency control) — an update writes a new version of the row instead of overwriting it, so readers see the version that matches their snapshot and **readers never block writers, nor writers readers**.

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
SHOW transaction_isolation;
COMMIT;
```

```text
BEGIN
 transaction_isolation
-----------------------
 repeatable read
(1 row)

COMMIT
```

### The lost update, and three fixes

Two sessions each sell one Desktop Tower at the same moment. Under Read Committed, a read-then-write done in the application loses an update:

```text
Session A                                          Session B
─────────                                          ─────────
SELECT stock FROM products WHERE product_id = 7;   SELECT stock FROM products WHERE product_id = 7;
  → 5                                                → 5
UPDATE products SET stock = 4 WHERE ...;           UPDATE products SET stock = 4 WHERE ...;
COMMIT;                                            COMMIT;   -- stock is 4, should be 3
```

Fix 1, the best when possible: **let the database do the arithmetic atomically.** A single `UPDATE` reads and writes the row under a row lock, and a condition in `WHERE` enforces the business rule:

```sql
UPDATE products SET stock = stock - 1 WHERE product_id = 7 AND stock >= 1 RETURNING stock;
```

```text
 stock
-------
     4
(1 row)

UPDATE 1
```

If the update affects zero rows, the item was out of stock — no separate check is needed.

Fix 2: **lock the row while deciding**, with `SELECT ... FOR UPDATE`. Other transactions that try to lock or modify the same row wait until this one commits:

```sql
BEGIN;
SELECT stock FROM products WHERE product_id = 7 FOR UPDATE;   -- row is now locked
UPDATE products SET stock = stock - 1 WHERE product_id = 7;
COMMIT;
```

```text
BEGIN
 stock
-------
     4
(1 row)

UPDATE 1
COMMIT
```

Fix 3: run at **Repeatable Read or Serializable** and retry the transaction when the database raises a serialization failure (SQLSTATE `40001`). This is the most general approach, and it requires retry logic in the application.

### Work queues with SKIP LOCKED

`FOR UPDATE SKIP LOCKED` skips rows that other transactions have locked, which turns a plain table into a safe job queue: many workers can each claim a different pending job with no double-processing and no waiting:

```sql
BEGIN;
SELECT order_id FROM orders
WHERE status = 'pending'
ORDER BY ordered_at
LIMIT 1
FOR UPDATE SKIP LOCKED;
-- ... process the order, update its status ...
COMMIT;
```

```text
BEGIN
 order_id
----------
       15
(1 row)

COMMIT
```

### Deadlocks

A **deadlock** occurs when transaction A holds a lock B needs while B holds a lock A needs. PostgreSQL detects this after a short wait and aborts one of them with an error. The prevention is to acquire locks in a consistent order — for example, always update rows sorted by ID — and to keep transactions short.

**Notes:**
- Keep transactions short. A transaction left open (an idle session after `BEGIN`) holds locks and prevents cleanup of old row versions, which bloats tables.
- Prefer atomic single statements (`SET x = x - 1 WHERE x >= 1`) over read-modify-write in application code.
- Read Committed is the default in PostgreSQL, Oracle, and SQL Server; MySQL's InnoDB defaults to Repeatable Read. Behavior at the same named level differs between engines, so check the documentation of yours.
- Any code running at Serializable must be prepared to retry. That is the price of the strongest guarantee.
- "What is ACID?" and "Explain isolation levels" are standard backend and data engineering interview questions. Anchor each level to the anomaly it prevents.

---

## 42. Query Plans and Performance

SQL is declarative, so performance work means understanding what the **query planner** decided and giving it what it needs to decide better. The planner estimates the cost of alternative plans using table statistics and picks the cheapest.

This section uses the million-row `events` table from [Section 39](#39-indexes), with indexes on `user_id` and `created_at`.

### EXPLAIN and EXPLAIN ANALYZE

`EXPLAIN` shows the chosen plan with the planner's **estimates**, without running the query. `EXPLAIN ANALYZE` runs the query and adds what **actually** happened. Always use `ANALYZE` when diagnosing (but inside a transaction you roll back for `INSERT`/`UPDATE`/`DELETE`, since it really executes them):

```sql
EXPLAIN
SELECT c.first_name, count(*)
FROM customers c
JOIN orders o USING (customer_id)
GROUP BY c.first_name;
```

```text
                                   QUERY PLAN
---------------------------------------------------------------------------------
 HashAggregate  (cost=47.22..49.22 rows=200 width=40)
   Group Key: c.first_name
   ->  Hash Join  (cost=19.23..42.12 rows=1020 width=32)
         Hash Cond: (o.customer_id = c.customer_id)
         ->  Seq Scan on orders o  (cost=0.00..20.20 rows=1020 width=4)
         ->  Hash  (cost=14.10..14.10 rows=410 width=36)
               ->  Seq Scan on customers c  (cost=0.00..14.10 rows=410 width=36)
(7 rows)
```

A plan is a tree read from the most indented nodes (executed first) outwards. Each node shows `cost=startup..total` in arbitrary units, the estimated `rows`, and the estimated row `width` in bytes.

Look at the estimates: 1,020 orders and 410 customers, for tables that hold 15 and 10 rows. These tables have never been analyzed, so the planner is falling back on default guesses. `ANALYZE` collects real statistics, and the estimates snap into line:

```sql
ANALYZE customers;
ANALYZE orders;

EXPLAIN
SELECT c.first_name, count(*)
FROM customers c
JOIN orders o USING (customer_id)
GROUP BY c.first_name;
```

```text
ANALYZE
ANALYZE
                                  QUERY PLAN
------------------------------------------------------------------------------
 HashAggregate  (cost=2.51..2.61 rows=10 width=13)
   Group Key: c.first_name
   ->  Hash Join  (cost=1.23..2.44 rows=15 width=5)
         Hash Cond: (o.customer_id = c.customer_id)
         ->  Seq Scan on orders o  (cost=0.00..1.15 rows=15 width=4)
         ->  Hash  (cost=1.10..1.10 rows=10 width=9)
               ->  Seq Scan on customers c  (cost=0.00..1.10 rows=10 width=9)
(7 rows)
```

### The nodes you will see

| Node | What it does | Good sign or warning |
|:--|:--|:--|
| Seq Scan | Reads the whole table | Fine for small tables or large fractions; a warning on big tables with selective filters |
| Index Scan | Walks an index, fetches matching rows | Good for few rows |
| Index Only Scan | Answers from the index alone | Best case |
| Bitmap Index/Heap Scan | Collects matches from an index, then fetches pages in order | Good for a moderate number of rows |
| Nested Loop | For each outer row, looks up inner rows | Great when the outer side is small and the inner side indexed |
| Hash Join | Builds a hash table of one side, probes it with the other | The workhorse for large unsorted joins |
| Merge Join | Merges two inputs sorted on the join key | Good when both sides come sorted |
| Sort, HashAggregate | Sorting and grouping | Watch for sorts spilling to disk ("external merge") |

### Estimates vs actual rows

The most useful thing `EXPLAIN ANALYZE` shows is the gap between **estimated** and **actual** rows. When they differ by orders of magnitude, the planner is choosing plans for data that does not exist, and the fix is usually better statistics (`ANALYZE table`), not a hint.

### Sargable predicates in practice

A predicate is **sargable** (Search ARGument ABLE) when it compares the bare indexed column to a value. Casting the column hides it from the index:

```sql
EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT count(*) FROM events WHERE created_at::date = '2026-03-01';
```

```text
                                QUERY PLAN
--------------------------------------------------------------------------
 Finalize Aggregate (actual rows=1 loops=1)
   ->  Gather (actual rows=3 loops=1)
         Workers Planned: 2
         Workers Launched: 2
         ->  Partial Aggregate (actual rows=1 loops=3)
               ->  Parallel Seq Scan on events (actual rows=1822 loops=3)
                     Filter: ((created_at)::date = '2026-03-01'::date)
                     Rows Removed by Filter: 331511
 Planning Time: 0.295 ms
 Execution Time: 108.047 ms
(10 rows)
```

```sql
EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT count(*) FROM events
WHERE created_at >= '2026-03-01' AND created_at < '2026-03-02';
```

```text
                                                                              QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate (actual rows=1 loops=1)
   ->  Bitmap Heap Scan on events (actual rows=5466 loops=1)
         Recheck Cond: ((created_at >= '2026-03-01 00:00:00+00'::timestamp with time zone) AND (created_at < '2026-03-02 00:00:00+00'::timestamp with time zone))
         Heap Blocks: exact=3819
         ->  Bitmap Index Scan on idx_events_created_at (actual rows=5466 loops=1)
               Index Cond: ((created_at >= '2026-03-01 00:00:00+00'::timestamp with time zone) AND (created_at < '2026-03-02 00:00:00+00'::timestamp with time zone))
 Planning Time: 0.267 ms
 Execution Time: 7.207 ms
(8 rows)
```

Same answer, same rows — the first scans a million rows and the second reads only the matching range of the index.

### Statistics, VACUUM, and bloat

The planner relies on statistics about each column — row counts, distinct values, most common values, histograms — gathered by `ANALYZE`. Because of MVCC, updates and deletes leave dead row versions behind; `VACUUM` reclaims them for reuse. PostgreSQL's **autovacuum** daemon runs both automatically, but after a bulk load or a large delete, run `ANALYZE` (or `VACUUM ANALYZE`) yourself so the next queries are planned with fresh numbers.

### A performance checklist

When a query is slow, work through these in order:

1. **Measure** with `EXPLAIN (ANALYZE, BUFFERS)`. Find the node where the time goes and compare estimated and actual rows.
2. **Missing index?** A sequential scan with a selective filter on a large table wants an index on the filter or join column.
3. **Non-sargable predicate?** Rewrite functions on columns into ranges, or add an expression index.
4. **Stale statistics?** Run `ANALYZE`.
5. **Fetching too much?** Select only needed columns, filter early, and paginate with keysets instead of large `OFFSET`s.
6. **Fan-out?** Aggregate before joining so the join processes fewer rows.
7. **Round trips?** Many tiny queries from an application loop (the N+1 problem, [Section 47](#47-orms-sqlalchemy-and-pandas)) are slower than one set-based query.
8. **Still slow?** Consider a materialized view, a summary table, partitioning ([Section 48](#48-data-engineering-warehouses-star-schemas-and-pipelines)), or moving the workload to an analytical engine.

The `pg_stat_statements` extension records the total time spent in every query shape across the server. Sorting it by total time is the fastest way to find which queries are worth optimizing in a real system.

**Notes:**
- `EXPLAIN` alone is a guess. `EXPLAIN ANALYZE` is evidence, and it actually executes the statement.
- Costs are in planner units, not milliseconds. Compare them only between plans for the same query.
- The biggest wins are almost always an index, a sargable rewrite, or not doing work at all (filtering earlier, fetching less). Server tuning comes much later.
- Optimize the queries that consume the most *total* time (frequency times duration), not the single slowest one.
- MySQL has `EXPLAIN ANALYZE` (8.0.18+), SQL Server shows graphical execution plans, and warehouses show query profiles. The concepts are the same — scans, joins, estimates versus actuals.

---

## 43. Functions, Procedures, and Triggers

Databases can run code. Used well, server-side logic enforces rules close to the data; used carelessly, it hides business logic where nobody looks for it.

### SQL functions

A function wraps an expression or query under a name. The simplest ones are written in plain SQL:

```sql
CREATE FUNCTION price_with_vat(net numeric, rate numeric DEFAULT 0.24)
RETURNS numeric
LANGUAGE sql
IMMUTABLE
RETURN round(net * (1 + rate), 2);

SELECT name, price, price_with_vat(price) AS gross, price_with_vat(price, 0.10) AS reduced
FROM products WHERE product_id <= 3;
```

```text
CREATE FUNCTION
        name         |  price  |  gross  | reduced
---------------------+---------+---------+---------
 Laptop Pro 14       | 1299.00 | 1610.76 | 1428.90
 Laptop Air 13       |  999.00 | 1238.76 | 1098.90
 Mechanical Keyboard |   89.90 |  111.48 |   98.89
(3 rows)
```

`IMMUTABLE` promises the result depends only on the arguments, which lets PostgreSQL use the function in expression indexes and optimize calls. `STABLE` means the result is constant within one statement (can read tables); `VOLATILE` (the default) means anything goes.

Functions can also return tables, which makes them parameterized views:

```sql
CREATE FUNCTION customer_orders(p_customer_id integer)
RETURNS TABLE (order_id integer, ordered_on date, total numeric)
LANGUAGE sql STABLE
AS $$
    SELECT o.order_id, o.ordered_at::date, sum(oi.quantity * oi.unit_price)
    FROM orders o JOIN order_items oi USING (order_id)
    WHERE o.customer_id = p_customer_id
    GROUP BY o.order_id, o.ordered_at
    ORDER BY o.ordered_at;
$$;

SELECT * FROM customer_orders(1);
```

```text
CREATE FUNCTION
 order_id | ordered_on |  total
----------+------------+---------
        1 | 2026-01-05 | 1228.99
        3 | 2026-01-28 |   89.90
        9 | 2026-03-30 |  143.00
(3 rows)
```

The `$$ ... $$` delimiters are **dollar quoting**: a way to write a string literal containing single quotes without escaping them.

### PL/pgSQL: procedural logic

PL/pgSQL adds variables, conditionals, loops, and exceptions:

```sql
CREATE FUNCTION customer_tier(p_customer_id integer)
RETURNS text
LANGUAGE plpgsql STABLE
AS $$
DECLARE
    v_total numeric;
BEGIN
    SELECT COALESCE(sum(oi.quantity * oi.unit_price), 0)
    INTO v_total
    FROM orders o JOIN order_items oi USING (order_id)
    WHERE o.customer_id = p_customer_id AND o.status <> 'cancelled';

    IF v_total >= 1400 THEN
        RETURN 'gold';
    ELSIF v_total > 0 THEN
        RETURN 'silver';
    ELSE
        RETURN 'none';
    END IF;
END;
$$;

SELECT first_name, customer_tier(customer_id) FROM customers WHERE customer_id IN (1, 2, 9);
```

```text
CREATE FUNCTION
 first_name | customer_tier
------------+---------------
 Alice      | gold
 Bruno      | silver
 Isla       | none
(3 rows)
```

### Procedures

A **procedure** (PostgreSQL 11+) is invoked with `CALL` and, unlike a function, can commit and roll back transactions internally, which suits long batch jobs:

```sql
CREATE PROCEDURE restock(p_product_id integer, p_units integer)
LANGUAGE plpgsql
AS $$
BEGIN
    IF p_units <= 0 THEN
        RAISE EXCEPTION 'units must be positive, got %', p_units;
    END IF;
    UPDATE products SET stock = stock + p_units WHERE product_id = p_product_id;
END;
$$;

CALL restock(5, 25);
SELECT name, stock FROM products WHERE product_id = 5;
CALL restock(5, -3);
```

```text
CREATE PROCEDURE
CALL
   name    | stock
-----------+-------
 USB-C Hub |    25
(1 row)

ERROR:  units must be positive, got -3
CONTEXT:  PL/pgSQL function restock(integer,integer) line 4 at RAISE
```

### Triggers

A **trigger** runs a function automatically when rows are inserted, updated, or deleted. The two classic uses are maintaining an `updated_at` timestamp and writing an audit log:

```sql
ALTER TABLE products ADD COLUMN updated_at timestamptz;

CREATE TABLE price_audit (
    product_id integer,
    old_price  numeric(10,2),
    new_price  numeric(10,2),
    changed_by text DEFAULT current_user
);

CREATE FUNCTION log_price_change() RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    NEW.updated_at := '2026-07-15 09:00+00';   -- would be now() in real code
    IF NEW.price IS DISTINCT FROM OLD.price THEN
        INSERT INTO price_audit (product_id, old_price, new_price)
        VALUES (OLD.product_id, OLD.price, NEW.price);
    END IF;
    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_products_price
BEFORE UPDATE ON products
FOR EACH ROW EXECUTE FUNCTION log_price_change();

UPDATE products SET price = 94.90 WHERE product_id = 3;
UPDATE products SET stock = 60    WHERE product_id = 3;   -- no price change, no audit row

SELECT product_id, price, stock, updated_at FROM products WHERE product_id = 3;
SELECT product_id, old_price, new_price FROM price_audit;
```

```text
ALTER TABLE
CREATE TABLE
CREATE FUNCTION
CREATE TRIGGER
UPDATE 1
UPDATE 1
 product_id | price | stock |       updated_at
------------+-------+-------+------------------------
          3 | 94.90 |    60 | 2026-07-15 09:00:00+00
(1 row)

 product_id | old_price | new_price
------------+-----------+-----------
          3 |     89.90 |     94.90
(1 row)
```

`NEW` and `OLD` hold the row after and before the change. A `BEFORE` trigger can modify `NEW` before it is written; an `AFTER` trigger sees the final row and is used for side effects.

### Where should logic live?

| Put it in the database when | Put it in the application when |
|:--|:--|
| It is a data integrity rule (constraints first, triggers if constraints cannot express it) | It is business workflow that changes often |
| It must hold no matter who writes the data | It needs external services (email, payments, APIs) |
| It is set-based and moving the data out would be slow | It benefits from the application's testing and deployment tools |

**Notes:**
- Reach for constraints before triggers. Triggers are invisible when reading application code and can surprise people debugging "impossible" data changes.
- Mark functions `IMMUTABLE` or `STABLE` when true: it enables indexing and better plans. Lying about it causes wrong results.
- Procedural dialects are the least portable part of SQL: PL/pgSQL (PostgreSQL), T-SQL (SQL Server), PL/SQL (Oracle), and MySQL's stored-program syntax all differ.
- Anonymous `DO $$ ... $$` blocks run PL/pgSQL once without creating a function, useful in migrations and one-off scripts.

---

## 44. Roles, Permissions, and Row-Level Security

Databases are shared resources, and access control decides who can read or change what. The guiding principle is **least privilege**: every user and application gets exactly the access it needs and nothing more.

### Roles

In PostgreSQL, users and groups are both **roles**. A role with `LOGIN` can connect; roles can be granted to other roles, forming groups:

```sql
CREATE ROLE analyst NOLOGIN;                         -- a group
CREATE ROLE dana LOGIN PASSWORD 'change-me' IN ROLE analyst;
CREATE ROLE app_backend LOGIN PASSWORD 'change-me';
```

```text
CREATE ROLE
CREATE ROLE
CREATE ROLE
```

### GRANT and REVOKE

Privileges are granted on objects: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, `REFERENCES`, `TRIGGER` on tables; `USAGE` on schemas and sequences; `EXECUTE` on functions; `CONNECT` on databases:

```sql
-- Analysts: read-only on everything in the public schema
GRANT USAGE ON SCHEMA public TO analyst;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analyst;

-- The application: read and write the tables it uses, but no DDL
GRANT SELECT, INSERT, UPDATE ON customers, orders, order_items TO app_backend;
GRANT SELECT ON products, categories TO app_backend;
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO app_backend;

-- Column-level: the app may read employees' names but not salaries
GRANT SELECT (employee_id, first_name, last_name) ON employees TO app_backend;
```

```text
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
```

Testing permissions by switching role:

```sql
SET ROLE dana;
SELECT count(*) FROM customers;
DELETE FROM customers WHERE customer_id = 10;
RESET ROLE;

SET ROLE app_backend;
SELECT salary FROM employees LIMIT 1;
RESET ROLE;
```

```text
SET
 count
-------
    10
(1 row)

ERROR:  permission denied for table customers
RESET
SET
ERROR:  permission denied for table employees
RESET
```

`GRANT ... ON ALL TABLES` covers only tables that exist now. `ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO analyst;` extends it to tables created later.

### Row-level security

**Row-level security** (RLS) restricts *which rows* a role can see or modify, enforced by the database on every query. It is the standard way to isolate tenants in a multi-tenant application, and to let each sales rep see only their own orders:

```sql
CREATE ROLE sales_rep NOLOGIN;
GRANT SELECT ON orders TO sales_rep;

ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
CREATE POLICY own_orders ON orders
    FOR SELECT TO sales_rep
    USING (employee_id = current_setting('app.employee_id')::int);

SET ROLE sales_rep;
SET app.employee_id = '8';          -- the application sets this per request
SELECT order_id, employee_id, status FROM orders ORDER BY order_id;
RESET ROLE;
```

```text
CREATE ROLE
GRANT
ALTER TABLE
CREATE POLICY
SET
SET
 order_id | employee_id |  status
----------+-------------+-----------
        2 |           8 | delivered
        5 |           8 | delivered
        8 |           8 | delivered
        9 |           8 | delivered
       12 |           8 | delivered
       15 |           8 | pending
(6 rows)

RESET
```

Only Tom's orders (employee 8) are visible, even though the query asked for all of them. Superusers and roles with `BYPASSRLS` always bypass RLS, and table owners bypass it unless it is forced with `ALTER TABLE ... FORCE ROW LEVEL SECURITY`.

### Access for automated clients and AI agents

Any automated client that generates SQL — a BI tool, a text-to-SQL assistant, an LLM agent with a database tool — should connect as a dedicated role that has only `SELECT` on an explicit list of tables or views, a `statement_timeout` to stop runaway queries, and ideally RLS or views that hide sensitive columns. Prompt instructions like "only run SELECT queries" are not a security boundary — the database permissions are:

```sql
CREATE ROLE llm_reader LOGIN PASSWORD 'change-me';
GRANT USAGE ON SCHEMA public TO llm_reader;
GRANT SELECT ON products, categories TO llm_reader;
ALTER ROLE llm_reader SET statement_timeout = '5s';
ALTER ROLE llm_reader SET default_transaction_read_only = on;
```

```text
CREATE ROLE
GRANT
GRANT
ALTER ROLE
ALTER ROLE
```

**Notes:**
- Never let applications connect as a superuser or as the table owner. A single SQL injection bug then becomes full control of the database.
- Grant privileges to group roles and add users to groups — managing grants per person does not scale.
- Views plus column privileges hide sensitive columns; RLS hides rows. Together they express most access rules.
- Credentials belong in environment variables or a secrets manager, never in source code or notebooks committed to Git.
- MySQL and SQL Server have the same `GRANT`/`REVOKE` model with different details; SQL Server also has row-level security. Warehouses add role hierarchies and column masking policies.

---

## 45. Import, Export, and Backup

### COPY: bulk loading and unloading

`COPY` moves data between a table and a file in bulk, far faster than `INSERT` statements (often 10 to 100 times). There are two variants:

- `COPY ... FROM '/path/file.csv'` runs **on the server** and reads files on the database server's filesystem (requires elevated privileges).
- `\copy ... FROM 'file.csv'` is a `psql` meta-command that reads files on **your machine** and streams them to the server. This is what you normally want.

Exporting a query result to CSV, then loading it back into a new table:

```sql
\copy (SELECT product_id, name, price FROM products WHERE category_id = 6) TO 'books.csv' WITH (FORMAT csv, HEADER)

CREATE TABLE books_import (product_id integer, name text, price numeric(10,2));
\copy books_import FROM 'books.csv' WITH (FORMAT csv, HEADER)
SELECT * FROM books_import;
```

```text
COPY 3
CREATE TABLE
COPY 3
 product_id |          name          | price
------------+------------------------+-------
          8 | SQL for Everyone       | 39.00
          9 | Python Deep Dive       | 49.00
         10 | Designing Data Systems | 55.00
(3 rows)
```

Useful options include `DELIMITER ';'`, `NULL 'NA'`, `ENCODING 'LATIN1'`, and `FORCE_NULL (col)`. By default `COPY` is all-or-nothing: one malformed row aborts the whole load (PostgreSQL 17 added an `ON_ERROR ignore` option to skip such rows), which is why pipelines usually load into a permissive **staging table** (all `text` columns) first, then validate and cast with `INSERT ... SELECT`.

### Loading from Python

Drivers expose `COPY` for fast loads from application code; [Section 46](#46-sql-from-python-drivers-parameters-and-injection) shows `psycopg`'s interface. pandas `to_sql` is convenient for small frames and slow for large ones unless it is configured to use `COPY` or multi-row inserts.

### Backups

A database without tested backups is a database waiting to be lost. PostgreSQL offers two families of backup:

| Kind | Tool | Captures | Use for |
|:--|:--|:--|:--|
| Logical | `pg_dump` / `pg_dumpall` | SQL or an archive of schema and data | Single databases, migrations between versions, copies for development |
| Physical | `pg_basebackup` plus WAL archiving | The data files and the write-ahead log | Whole clusters, **point-in-time recovery** (restoring to any moment) |

```bash
# Logical backup of one database in the compressed custom format
pg_dump -h localhost -U postgres -d shop -Fc -f shop.dump

# Schema only (useful for code review and documentation)
pg_dump -h localhost -U postgres -d shop --schema-only -f shop_schema.sql

# Restore into a new, empty database
createdb -h localhost -U postgres shop_restored
pg_restore -h localhost -U postgres -d shop_restored shop.dump
```

Managed services (RDS, Cloud SQL, Azure, Supabase, Neon) run physical backups and point-in-time recovery for you — your job is to know the retention period and to practice restoring.

**Notes:**
- Use `\copy` from `psql` (or the driver's `COPY` API) for bulk loads. It is the fastest way to get data into PostgreSQL.
- Load into staging tables first, validate, then move clean rows into the real tables. One bad row should not block a whole load silently or corrupt a table.
- A backup you have never restored is a hope, not a backup. Test restores regularly.
- Other engines: MySQL `LOAD DATA INFILE` and `mysqldump`; SQL Server `BULK INSERT` and `bcp`; warehouses load from cloud storage (`COPY INTO` in Snowflake, load jobs in BigQuery), usually from Parquet files.

---

## 46. SQL from Python: Drivers, Parameters, and Injection

Most SQL in production is not typed into `psql` — it is sent by programs. Python talks to databases through **drivers** that follow the DB-API 2.0 standard (PEP 249), so the pattern is the same for every database: connect, get a cursor, execute SQL with parameters, fetch results, commit.

| Database | Common Python driver | Placeholder style |
|:--|:--|:--|
| PostgreSQL | `psycopg` (version 3), `asyncpg` for asyncio | `%s`, `%(name)s` |
| SQLite | `sqlite3` (standard library) | `?`, `:name` |
| MySQL | `mysql-connector-python`, `PyMySQL` | `%s` |
| SQL Server | `pyodbc` | `?` |
| DuckDB | `duckdb` | `?`, `$name` |

Install the PostgreSQL driver with `pip install "psycopg[binary]"`.

### Connecting and querying

```python
import psycopg

DSN = "postgresql://postgres:secret@localhost:5432/shop"   # in real code: os.environ["DATABASE_URL"]

with psycopg.connect(DSN) as conn:                 # commits on success, rolls back on error
    with conn.cursor() as cur:
        cur.execute("SELECT name, price FROM products WHERE category_id = %s ORDER BY price", (3,))
        for name, price in cur.fetchall():
            print(f"{name:<15} {price:>8}  {type(price).__name__}")
```

```text
Laptop Air 13     999.00  Decimal
Laptop Pro 14    1299.00  Decimal
```

Types map automatically: `numeric` arrives as Python's `Decimal` (exact, so money stays exact), `timestamptz` as a timezone-aware `datetime`, `jsonb` as `dict`, arrays as `list`, and `NULL` as `None`. `fetchone()`, `fetchmany(n)`, and iterating over the cursor are alternatives to `fetchall()` for large results.

### Parameters and SQL injection

**Never build SQL by pasting values into the string** (f-strings, `+`, `.format`). If a value comes from a user, they can change the meaning of your query. This is **SQL injection**, consistently one of the most exploited vulnerabilities in web applications:

```python
import psycopg

DSN = "postgresql://postgres:secret@localhost:5432/shop"
user_input = "nobody@example.com' OR '1'='1"      # a malicious "email"

with psycopg.connect(DSN) as conn:
    unsafe = f"SELECT count(*) FROM customers WHERE email = '{user_input}'"
    print("The query that actually ran:", unsafe)
    print("Unsafe match count:", conn.execute(unsafe).fetchone()[0])

    safe = conn.execute("SELECT count(*) FROM customers WHERE email = %s", (user_input,))
    print("Parameterized match count:", safe.fetchone()[0])
```

```text
The query that actually ran: SELECT count(*) FROM customers WHERE email = 'nobody@example.com' OR '1'='1'
Unsafe match count: 10
Parameterized match count: 0
```

The injected `OR '1'='1'` made the unsafe query match every customer. With a parameter, the driver sends the value separately from the SQL text (or escapes it correctly), so it is only ever treated as data — an odd-looking email that matches nobody. The same attack can append `; DROP TABLE customers` or read other tables.

Parameters substitute **values** only. They cannot supply table names, column names, or keywords. When identifiers must be dynamic — for example, a sort column chosen in a UI — validate them against an allow-list or use the driver's identifier quoting:

```python
import psycopg
from psycopg import sql

DSN = "postgresql://postgres:secret@localhost:5432/shop"
sort_column = "price"                     # must come from an allow-list, never raw user input
assert sort_column in {"name", "price", "stock"}

query = sql.SQL("SELECT name FROM products ORDER BY {} DESC LIMIT 2").format(sql.Identifier(sort_column))
with psycopg.connect(DSN) as conn:
    print(query.as_string(conn))
    print(conn.execute(query).fetchall())
```

```text
SELECT name FROM products ORDER BY "price" DESC LIMIT 2
[('Desktop Tower',), ('Laptop Pro 14',)]
```

### Transactions from Python

`psycopg` opens a transaction implicitly on the first statement; the connection's `with` block commits at the end or rolls back if an exception escapes. `conn.transaction()` creates an explicit (possibly nested, via savepoints) transaction block:

```python
import psycopg

DSN = "postgresql://postgres:secret@localhost:5432/shop"
with psycopg.connect(DSN) as conn:
    try:
        with conn.transaction():
            conn.execute("UPDATE products SET stock = stock - 1 WHERE product_id = 1")
            conn.execute("UPDATE products SET price = -1 WHERE product_id = 1")   # violates CHECK
    except psycopg.errors.CheckViolation as e:
        print("Rolled back:", e.diag.constraint_name)
    print("Stock unchanged:", conn.execute("SELECT stock FROM products WHERE product_id = 1").fetchone()[0])
```

```text
Rolled back: products_price_check
Stock unchanged: 15
```

### Bulk operations

`executemany` runs one statement for many parameter sets, and `COPY` streams data in bulk, which is the fastest way to load rows from Python:

```python
import psycopg

DSN = "postgresql://postgres:secret@localhost:5432/shop"
rows = [(f"user{i}@example.com", i % 5) for i in range(10_000)]

with psycopg.connect(DSN) as conn:
    conn.execute("CREATE TEMP TABLE clicks (email text, clicks int)")
    with conn.cursor() as cur:
        with cur.copy("COPY clicks (email, clicks) FROM STDIN") as copy:
            for row in rows:
                copy.write_row(row)
    print(conn.execute("SELECT count(*), sum(clicks) FROM clicks").fetchone())
```

```text
(10000, 20000)
```

### Rows as dictionaries, and SQLite

Row factories change the shape of fetched rows. And because every driver follows DB-API, the same code shape works for SQLite from the standard library, which is perfect for tests and small tools:

```python
import sqlite3

conn = sqlite3.connect(":memory:")          # or a file path, e.g. "app.db"
conn.row_factory = sqlite3.Row
conn.execute("CREATE TABLE notes (id INTEGER PRIMARY KEY, body TEXT NOT NULL)")
conn.executemany("INSERT INTO notes (body) VALUES (?)", [("first",), ("second",)])
conn.commit()
for row in conn.execute("SELECT id, body FROM notes WHERE id >= ?", (1,)):
    print(dict(row))
conn.close()
```

```text
{'id': 1, 'body': 'first'}
{'id': 2, 'body': 'second'}
```

### Connection pooling

Opening a database connection is expensive — tens of milliseconds and a server process in PostgreSQL. Web applications therefore use a **connection pool**: a set of open connections that requests borrow and return. Use `psycopg_pool.ConnectionPool`, SQLAlchemy's built-in pool, or a server-side pooler such as PgBouncer when many application instances share one database. PostgreSQL's `max_connections` is typically in the low hundreds, so thousands of clients must share through a pool.

**Notes:**
- Parameterize every value, every time, including values you think are safe. Injection bugs come from the one query someone assumed was internal.
- Placeholders differ by driver (`%s`, `?`, `:name`), but the rule is universal: values go in the parameters argument, never in the SQL string.
- Keep credentials out of code: read the connection URL from an environment variable or a secrets manager.
- Close what you open. Context managers (`with`) guarantee connections and cursors are closed and transactions ended even when errors occur.
- For async frameworks (FastAPI, aiohttp), use `psycopg`'s async API or `asyncpg`, so database waits do not block the event loop.

---

## 47. ORMs, SQLAlchemy, and pandas

### The layers

Python code reaches SQL at three levels of abstraction:

| Layer | Example | You write | Best for |
|:--|:--|:--|:--|
| Driver | `psycopg`, `sqlite3` | Raw SQL strings with parameters | Scripts, performance-critical paths, full control |
| Query builder | SQLAlchemy Core | Python expressions that compile to SQL | Dynamic queries, portability across databases |
| ORM | SQLAlchemy ORM, Django ORM | Classes and objects mapped to tables | Application code with rich domain models |

**SQLAlchemy** is the standard Python toolkit and provides both Core and ORM. Even with an ORM, knowing SQL remains essential: you need it to understand what the ORM generates, to diagnose slow pages, and to write the queries the ORM expresses badly.

### SQLAlchemy engine and text queries

The **engine** manages a connection pool. `text()` runs raw SQL with named, bound parameters:

```python
from sqlalchemy import create_engine, text

engine = create_engine("postgresql+psycopg://postgres:secret@localhost:5432/shop")

with engine.connect() as conn:
    result = conn.execute(
        text("SELECT first_name, country FROM customers WHERE country = :country ORDER BY first_name"),
        {"country": "Canada"},
    )
    for row in result:
        print(row.first_name, row.country)
```

```text
Chloe Canada
Emma Canada
```

### SQLAlchemy Core: SQL as Python expressions

Core builds queries from Python objects, which is valuable when filters are assembled dynamically (only add a `WHERE` when the user chose a filter) and when the same code must target several databases:

```python
from sqlalchemy import create_engine, MetaData, Table, select, func

engine = create_engine("postgresql+psycopg://postgres:secret@localhost:5432/shop")
meta = MetaData()
orders = Table("orders", meta, autoload_with=engine)      # reflect the existing table

stmt = (
    select(orders.c.status, func.count().label("n"))
    .group_by(orders.c.status)
    .order_by(func.count().desc())
)
print(stmt)          # the SQL it compiles to
print()
with engine.connect() as conn:
    print(conn.execute(stmt).all())
```

```text
SELECT orders.status, count(*) AS n
FROM orders GROUP BY orders.status ORDER BY count(*) DESC

[('delivered', 11), ('shipped', 2), ('cancelled', 1), ('pending', 1)]
```

### The ORM, and the N+1 problem

The ORM maps classes to tables and rows to objects, and relationships to attributes. Its most famous pitfall is the **N+1 query problem**: loading N parent objects with one query, then triggering one more query per parent when a relationship is accessed in a loop:

```python
from sqlalchemy import create_engine, ForeignKey, event, select
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship, Session, selectinload

class Base(DeclarativeBase):
    pass

class Customer(Base):
    __tablename__ = "customers"
    customer_id: Mapped[int] = mapped_column(primary_key=True)
    first_name: Mapped[str]
    orders: Mapped[list["Order"]] = relationship(back_populates="customer")

class Order(Base):
    __tablename__ = "orders"
    order_id: Mapped[int] = mapped_column(primary_key=True)
    customer_id: Mapped[int] = mapped_column(ForeignKey("customers.customer_id"))
    status: Mapped[str]
    customer: Mapped[Customer] = relationship(back_populates="orders")

engine = create_engine("postgresql+psycopg://postgres:secret@localhost:5432/shop")
query_count = 0

@event.listens_for(engine, "before_cursor_execute")
def count_queries(*args):
    global query_count
    query_count += 1

with Session(engine) as session:
    query_count = 0
    for c in session.scalars(select(Customer)):              # 1 query for customers...
        _ = len(c.orders)                                    # ...plus 1 per customer
    print("Lazy loading:  ", query_count, "queries")

    query_count = 0
    stmt = select(Customer).options(selectinload(Customer.orders))
    for c in session.scalars(stmt):
        _ = len(c.orders)
    print("selectinload:  ", query_count, "queries")
```

```text
Lazy loading:   11 queries
selectinload:   2 queries
```

Eleven round trips for ten customers becomes two, regardless of how many customers there are. The fix is **eager loading** (`selectinload`, `joinedload`) whenever you know you will touch the relationship, and the way to catch N+1 is to log or count the SQL your code emits (`create_engine(..., echo=True)`).

### pandas

pandas reads query results straight into a DataFrame. Push filtering and aggregation into SQL, where the database can use indexes and moves less data, and use pandas for what follows:

```python
import pandas as pd
from sqlalchemy import create_engine, text

engine = create_engine("postgresql+psycopg://postgres:secret@localhost:5432/shop")

df = pd.read_sql(
    text("""
        SELECT date_trunc('month', o.ordered_at)::date AS month,
               sum(oi.quantity * oi.unit_price)::float AS revenue
        FROM orders o JOIN order_items oi USING (order_id)
        WHERE o.status <> :excluded
        GROUP BY 1 ORDER BY 1
    """),
    engine,
    params={"excluded": "cancelled"},
)
df["share"] = (df["revenue"] / df["revenue"].sum()).round(3)
print(df.to_string(index=False))

df.to_sql("monthly_revenue_report", engine, if_exists="replace", index=False)
with engine.connect() as conn:
    print(conn.execute(text("SELECT count(*) FROM monthly_revenue_report")).scalar())
```

```text
     month  revenue  share
2026-01-01  1445.89  0.181
2026-02-01  1601.98  0.200
2026-03-01   436.00  0.055
2026-04-01  1478.87  0.185
2026-05-01  1293.00  0.162
2026-06-01  1743.90  0.218
6
```

Casting `numeric` to `float` in the query (`::float`) gives pandas a float column — without it, values arrive as `Decimal` objects in an `object` column, which is exact but slow for numeric work. `to_sql` writes a DataFrame to a table; for large frames, pass `method="multi"` and a `chunksize`, or use `COPY`.

### Migrations

Applications change their schema over time through **migrations**: numbered scripts that move the schema forward (and optionally back). Alembic is SQLAlchemy's migration tool and Django has its own — both can autogenerate a migration from model changes, which you then review, because generated DDL on a large table can lock it for a long time ([Section 35](#35-creating-and-altering-tables)).

**Notes:**
- ORMs are productive for everyday create, read, update, delete work. Drop to SQL (`text()` or a view) for reporting queries, bulk operations, and anything performance-critical.
- Watch the SQL your ORM generates. `echo=True` in development is the fastest way to spot N+1 patterns and accidental full-table loads.
- `pd.read_sql` on `SELECT * FROM big_table` pulls everything into memory. Aggregate in SQL first, or stream in chunks with `chunksize=`.
- Use `sqlalchemy.text()` with bound parameters, not f-strings, even inside pandas calls. Injection rules do not change because the result becomes a DataFrame.

---

## 48. Data Engineering: Warehouses, Star Schemas, and Pipelines

Data engineers move data from the systems that create it — OLTP databases, APIs, event streams — into systems built to analyze it (warehouses and lakehouses), keeping it correct, fresh, and cheap to query along the way. Nearly every step is SQL.

### OLTP vs OLAP, in practice

| | OLTP (the shop's PostgreSQL) | OLAP (the warehouse) |
|:--|:--|:--|
| Workload | Many small reads and writes | Few large scans and aggregations |
| Schema | Normalized (3NF) | Denormalized (star schema, wide tables) |
| Storage | Row-oriented | Column-oriented |
| Data | Current state | History, often years of it |
| Typical query | "Fetch order 42" | "Revenue by region by month since 2020" |
| Users | The application | Analysts, dashboards, ML pipelines |

Running heavy analytics directly on the production database competes with customers for the same resources, so analytical data is copied out, reshaped, and served from a separate system.

### Dimensional modeling: the star schema

A **star schema** has one central **fact table** of measurable events (sales, clicks, payments) at a declared **grain**, surrounded by **dimension tables** that describe the who, what, where, and when. Facts are narrow and long; dimensions are wide and short:

```text
               dim_date                 dim_product
                   │                        │
                   └──────┐          ┌──────┘
                        fact_sales (grain: one order line)
                   ┌──────┘          └──────┐
                   │                        │
             dim_customer            (more dimensions...)
```

Building one from the shop, starting with a date dimension (one row per day, with every attribute a report might group by):

```sql
CREATE SCHEMA dw;

CREATE TABLE dw.dim_date AS
SELECT to_char(d, 'YYYYMMDD')::int         AS date_key,
       d::date                             AS full_date,
       extract(year FROM d)::int           AS year,
       extract(quarter FROM d)::int        AS quarter,
       extract(month FROM d)::int          AS month,
       to_char(d, 'Mon')                   AS month_name,
       extract(isodow FROM d) IN (6, 7)    AS is_weekend
FROM generate_series('2026-01-01'::date, '2026-12-31'::date, interval '1 day') AS d;

CREATE TABLE dw.dim_product AS
WITH RECURSIVE tree AS (                     -- flatten the category tree once, here
    SELECT category_id, name AS department FROM categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.category_id, t.department FROM categories c JOIN tree t ON c.parent_id = t.category_id
)
SELECT p.product_id AS product_key, p.name AS product, cat.name AS category,
       tree.department, p.attributes ->> 'brand' AS brand
FROM products p
JOIN categories cat USING (category_id)
JOIN tree USING (category_id);

CREATE TABLE dw.dim_customer AS
SELECT customer_id AS customer_key, first_name || ' ' || last_name AS customer,
       COALESCE(city, 'Unknown') AS city, country
FROM customers;

CREATE TABLE dw.fact_sales AS
SELECT to_char(o.ordered_at, 'YYYYMMDD')::int AS date_key,
       o.customer_id                          AS customer_key,
       oi.product_id                          AS product_key,
       o.order_id,
       oi.quantity,
       oi.quantity * oi.unit_price            AS revenue
FROM orders o JOIN order_items oi USING (order_id)
WHERE o.status <> 'cancelled';

SELECT count(*) AS fact_rows FROM dw.fact_sales;
```

```text
CREATE SCHEMA
SELECT 365
SELECT 12
SELECT 10
SELECT 25
 fact_rows
-----------
        25
(1 row)
```

Every analytical question is now the same shape: filter and group by dimension attributes, sum the facts:

```sql
SELECT d.quarter, p.department, c.country, sum(f.revenue) AS revenue
FROM dw.fact_sales f
JOIN dw.dim_date d     USING (date_key)
JOIN dw.dim_product p  USING (product_key)
JOIN dw.dim_customer c USING (customer_key)
GROUP BY d.quarter, p.department, c.country
ORDER BY d.quarter, revenue DESC
LIMIT 6;
```

```text
 quarter | department  | country  | revenue
---------+-------------+----------+---------
       1 | Electronics | UK       | 1517.89
       1 | Electronics | Canada   | 1103.98
       1 | Electronics | Germany  |  498.00
       1 | Books       | UK       |  143.00
       1 | Books       | Portugal |  127.00
       1 | Books       | Canada   |   94.00
(6 rows)
```

The `department` column (the top-level category) is denormalized — the category tree was flattened into the dimension once, with a recursive CTE, so no report ever needs one. A **snowflake schema** normalizes dimensions into sub-dimensions instead — it saves space and costs joins, and is less common today.

### Slowly changing dimensions

Dimension attributes change: a customer moves city. Should last year's sales be reported under the old city or the new one? **Slowly changing dimension** (SCD) types are the standard answers:

| Type | Behavior | History |
|:--|:--|:--|
| Type 1 | Overwrite the attribute | Lost; all facts show the current value |
| Type 2 | Close the current row and insert a new version with validity dates | Kept; each fact joins to the version valid at the time |
| Type 3 | Keep a `previous_value` column | Only one prior value |

Type 2 is the one worth knowing in detail:

```sql
CREATE TABLE dw.dim_customer_scd (
    customer_sk  integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,   -- surrogate key per version
    customer_id  integer NOT NULL,                                    -- the business key
    city         text    NOT NULL,
    valid_from   date    NOT NULL,
    valid_to     date    NOT NULL DEFAULT '9999-12-31',
    is_current   boolean NOT NULL DEFAULT true
);
INSERT INTO dw.dim_customer_scd (customer_id, city, valid_from)
SELECT customer_id, COALESCE(city, 'Unknown'), signup_date FROM customers;

-- Bruno moves from Lisbon to Porto on 2026-04-01: close the old version, open a new one
UPDATE dw.dim_customer_scd
SET valid_to = '2026-04-01', is_current = false
WHERE customer_id = 2 AND is_current;

INSERT INTO dw.dim_customer_scd (customer_id, city, valid_from)
VALUES (2, 'Porto', '2026-04-01');

-- Each order is attributed to the city Bruno lived in when he ordered
SELECT o.order_id, o.ordered_at::date, d.city
FROM orders o
JOIN dw.dim_customer_scd d
  ON d.customer_id = o.customer_id
 AND o.ordered_at >= d.valid_from AND o.ordered_at < d.valid_to
WHERE o.customer_id = 2
ORDER BY o.ordered_at;
```

```text
CREATE TABLE
INSERT 0 10
UPDATE 1
INSERT 0 1
 order_id | ordered_at |  city
----------+------------+--------
        2 | 2026-01-12 | Lisbon
       11 | 2026-04-25 | Porto
(2 rows)
```

The half-open validity range (`>= valid_from AND < valid_to`) guarantees exactly one version matches any moment. In a real pipeline, the close-and-insert pair runs in one transaction or one `MERGE`, and tools like dbt snapshots generate it for you.

### Incremental, idempotent loads

Reloading everything every night stops scaling. **Incremental loads** process only rows that changed since the last run, tracked by a **watermark** (the highest `updated_at` or ID already loaded). Combined with an upsert, the load is **idempotent**: rerunning it after a failure never duplicates data.

```sql
CREATE TABLE dw.etl_watermark (source text PRIMARY KEY, loaded_until timestamptz NOT NULL);
INSERT INTO dw.etl_watermark VALUES ('orders', '2026-01-01');

CREATE TABLE dw.orders_copy (LIKE orders INCLUDING ALL);

-- One incremental run: copy orders newer than the watermark, then advance it
WITH batch AS (
    SELECT * FROM orders
    WHERE ordered_at > (SELECT loaded_until FROM dw.etl_watermark WHERE source = 'orders')
      AND ordered_at <= '2026-03-31'                      -- this run's upper bound
),
upserted AS (
    INSERT INTO dw.orders_copy SELECT * FROM batch
    ON CONFLICT (order_id) DO UPDATE SET status = EXCLUDED.status
    RETURNING 1
)
UPDATE dw.etl_watermark
SET loaded_until = COALESCE((SELECT max(ordered_at) FROM batch), loaded_until)
WHERE source = 'orders'
RETURNING loaded_until, (SELECT count(*) FROM upserted) AS rows_loaded;
```

```text
CREATE TABLE
INSERT 0 1
CREATE TABLE
      loaded_until      | rows_loaded
------------------------+-------------
 2026-03-30 21:05:00+00 |           9
(1 row)

UPDATE 1
```

Running the same statement again loads nothing, because the watermark moved. If the job crashed halfway, the transaction rolled back both the rows and the watermark, so the retry starts cleanly. Two refinements matter in practice: a real watermark column should be an `updated_at` that changes on every modification (not a creation time), and late-arriving rows are handled by re-reading a small overlap window, which the upsert makes harmless.

### Partitioning

**Partitioning** splits one logical table into physical pieces by a key, usually time. Queries that filter on the key skip irrelevant partitions (**partition pruning**), and old data is dropped instantly by detaching a partition instead of running a huge `DELETE`:

```sql
CREATE TABLE dw.events (
    event_id   bigint,
    user_id    integer,
    created_at timestamptz NOT NULL
) PARTITION BY RANGE (created_at);

CREATE TABLE dw.events_2026_01 PARTITION OF dw.events FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
CREATE TABLE dw.events_2026_02 PARTITION OF dw.events FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');
CREATE TABLE dw.events_2026_03 PARTITION OF dw.events FOR VALUES FROM ('2026-03-01') TO ('2026-04-01');

INSERT INTO dw.events
SELECT g, g % 100, '2026-01-01'::timestamptz + (g % 89) * interval '1 day'
FROM generate_series(1, 9000) AS g;

EXPLAIN (COSTS OFF)
SELECT count(*) FROM dw.events WHERE created_at >= '2026-02-10' AND created_at < '2026-02-20';
```

```text
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
INSERT 0 9000
                                                                         QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate
   ->  Seq Scan on events_2026_02 events
         Filter: ((created_at >= '2026-02-10 00:00:00+00'::timestamp with time zone) AND (created_at < '2026-02-20 00:00:00+00'::timestamp with time zone))
(3 rows)
```

Only the February partition is scanned. Warehouses expose the same idea as partitioned and clustered tables (BigQuery), micro-partitions with clustering keys (Snowflake), and partitioned Parquet directories in data lakes.

### Data quality checks

Pipelines fail silently: a source changes a column, a join fans out, a load runs twice. Data tests catch this by asserting properties of the data. Each test is a query that looks for **violating rows** and passes when it finds none; here several tests are combined into one report of failure counts:

```sql
SELECT 'orders: duplicate order_id' AS test, count(*) AS failures
FROM (SELECT order_id FROM orders GROUP BY order_id HAVING count(*) > 1) t
UNION ALL
SELECT 'order_items: orphaned lines', count(*)
FROM order_items oi WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.order_id = oi.order_id)
UNION ALL
SELECT 'customers: missing city', count(*)
FROM customers WHERE city IS NULL
UNION ALL
SELECT 'orders: unexpected status', count(*)
FROM orders WHERE status NOT IN ('pending', 'shipped', 'delivered', 'cancelled')
UNION ALL
SELECT 'fact_sales: revenue mismatch vs source', count(*)
FROM (SELECT (SELECT sum(revenue) FROM dw.fact_sales)
           - (SELECT sum(oi.quantity * oi.unit_price) FROM order_items oi
              JOIN orders o USING (order_id) WHERE o.status <> 'cancelled') AS diff) t
WHERE diff <> 0;
```

```text
                  test                  | failures
----------------------------------------+----------
 orders: duplicate order_id             |        0
 order_items: orphaned lines            |        0
 customers: missing city                |        2
 orders: unexpected status              |        0
 fact_sales: revenue mismatch vs source |        0
(5 rows)
```

The standard checks are uniqueness, not-null, referential integrity, accepted values, row-count and sum **reconciliation** against the source, and **freshness** (the newest row is recent enough). The missing-city result here is a real finding — a warning to investigate, or a known and documented gap.

### ELT, dbt, and orchestration

Modern pipelines are usually **ELT**: extract raw data, load it unchanged into the warehouse, then transform it there with SQL (the older ETL order transformed before loading). **dbt** is the dominant tool for the transform step — each model is a `SELECT` statement in a file, dbt resolves dependencies between models, materializes them as views, tables, or incremental tables, and runs data tests like the ones above declared in YAML. An **orchestrator** (Airflow, Dagster, Prefect) schedules the whole flow and retries failures. **Change data capture** (CDC), reading the database's write-ahead log with tools like Debezium or PostgreSQL logical replication, streams every insert, update, and delete to the warehouse without querying the source tables at all.

**Notes:**
- Declare the grain of every fact table in one sentence ("one row per order line") before writing it. Most warehouse bugs are grain bugs.
- Use surrogate keys in dimensions when you keep history (SCD Type 2); the business key alone is no longer unique.
- Make every load idempotent (upserts, `MERGE`, or delete-and-reinsert of a whole partition), so retries are always safe.
- Test data, not just code: uniqueness, nulls, relationships, accepted values, reconciliation, and freshness, on every run.
- Interview favorites for data engineers: star vs snowflake schema, SCD types, OLTP vs OLAP, idempotency, partitioning, and "how would you load this incrementally?"

---

## 49. Analytical Engines: DuckDB and Cloud Warehouses

### Why column stores are fast for analytics

A row store keeps each row's values together on disk, which is ideal for fetching or updating one whole record. A **column store** keeps each column's values together:

```text
Row store (PostgreSQL)                    Column store (DuckDB, BigQuery, Snowflake)
[1, Alice, UK, 2025-01-05]                customer_id: [1, 2, 3, ...]
[2, Bruno, Portugal, 2025-01-18]          first_name:  [Alice, Bruno, Chloe, ...]
[3, Chloe, Canada, 2025-02-02]            country:     [UK, Portugal, Canada, ...]
                                          signup_date: [2025-01-05, 2025-01-18, ...]
```

A query like `SELECT country, count(*) FROM customers GROUP BY country` needs one column out of perhaps fifty, so a column store reads only that one, while a row store must read every row in full. Values in a column are similar, so they compress extremely well, and engines process them in vectorized batches. The result is aggregation over billions of rows in seconds, at the cost of slow single-row updates.

### DuckDB: a warehouse in a Python import

**DuckDB** is an embedded analytical database, "SQLite for analytics": `pip install duckdb`, no server, and it queries CSV, Parquet, and JSON files and pandas or Polars DataFrames directly with a PostgreSQL-like dialect. It has become a standard tool for data scientists and for local data engineering work:

```python
import duckdb
import pandas as pd
from sqlalchemy import create_engine

# Export two tables from PostgreSQL to local files to simulate a data lake
engine = create_engine("postgresql+psycopg://postgres:secret@localhost:5432/shop")
pd.read_sql("SELECT * FROM orders", engine).to_csv("orders.csv", index=False)
items = pd.read_sql("SELECT order_id, product_id, quantity, unit_price::float AS unit_price FROM order_items", engine)

# Query a CSV file and a DataFrame together, as if both were tables
result = duckdb.sql("""
    SELECT strftime(o.ordered_at, '%Y-%m') AS month,
           count(DISTINCT o.order_id)      AS orders,
           round(sum(i.quantity * i.unit_price), 2) AS revenue
    FROM 'orders.csv' AS o
    JOIN items AS i ON i.order_id = o.order_id      -- 'items' is the pandas DataFrame
    WHERE o.status <> 'cancelled'
    GROUP BY ALL                                    -- group by every non-aggregated column
    ORDER BY month
""")
print(result)

# Convert to Parquet, the standard columnar file format of data lakes
duckdb.sql("COPY (SELECT * FROM 'orders.csv') TO 'orders.parquet' (FORMAT parquet)")
print(duckdb.sql("SELECT count(*) AS rows, min(ordered_at) AS first_order FROM 'orders.parquet'"))
```

```text
┌─────────┬────────┬─────────┐
│  month  │ orders │ revenue │
│ varchar │ int64  │ double  │
├─────────┼────────┼─────────┤
│ 2026-01 │      3 │ 1445.89 │
│ 2026-02 │      2 │ 1601.98 │
│ 2026-03 │      3 │   436.0 │
│ 2026-04 │      2 │ 1478.87 │
│ 2026-05 │      2 │  1293.0 │
│ 2026-06 │      2 │  1743.9 │
└─────────┴────────┴─────────┘

┌───────┬──────────────────────────┐
│ rows  │       first_order        │
│ int64 │ timestamp with time zone │
├───────┼──────────────────────────┤
│    15 │ 2026-01-05 09:15:00+00   │
└───────┴──────────────────────────┘
```

DuckDB also includes conveniences that cloud warehouses share, notably `QUALIFY` (filter on window functions without a subquery) and `EXCLUDE` (all columns except some):

```python
import duckdb

print(duckdb.sql("""
    SELECT * EXCLUDE (employee_id)
    FROM 'orders.parquet'
    QUALIFY row_number() OVER (PARTITION BY customer_id ORDER BY ordered_at DESC) = 1
    ORDER BY customer_id
    LIMIT 4
"""))
```

```text
┌──────────┬─────────────┬──────────────────────────┬───────────┐
│ order_id │ customer_id │        ordered_at        │  status   │
│  int64   │    int64    │ timestamp with time zone │  varchar  │
├──────────┼─────────────┼──────────────────────────┼───────────┤
│        9 │           1 │ 2026-03-30 21:05:00+00   │ delivered │
│       11 │           2 │ 2026-04-25 17:25:00+00   │ delivered │
│       15 │           3 │ 2026-06-20 18:00:00+00   │ pending   │
│       13 │           4 │ 2026-05-19 15:35:00+00   │ shipped   │
└──────────┴─────────────┴──────────────────────────┴───────────┘
```

Use DuckDB for exploratory analysis of files, notebook work that outgrows pandas' memory or speed, local pipeline development, and as a fast engine inside Python tools. Do not use it as an application's transactional database — that is PostgreSQL's job.

### Cloud warehouses

The same SQL skills run the major cloud warehouses. What changes is the **cost model** and the **physical design levers**:

| Engine | Billing | Main physical design lever | Notable SQL features |
|:--|:--|:--|:--|
| BigQuery | Bytes scanned per query (on-demand) or reserved slots | Partitioned and clustered tables | `QUALIFY`, `STRUCT` and `ARRAY` columns, `UNNEST`, `SAFE_` functions |
| Snowflake | Compute time of virtual warehouses | Automatic micro-partitions, optional clustering keys | `QUALIFY`, `VARIANT` for JSON, time travel (`AT (OFFSET => ...)`), zero-copy cloning |
| Redshift | Cluster size or serverless compute | Distribution and sort keys | PostgreSQL-derived syntax |
| Databricks SQL | Compute time | Delta tables, partitioning, liquid clustering | Spark SQL, `MERGE INTO`, notebooks next to SQL |

Habits that save money and time in any warehouse:

- **Never `SELECT *` on big tables.** In a column store you pay (in bytes or seconds) for every column you touch. `LIMIT` does not reduce what is scanned in BigQuery.
- **Filter on the partition column** (usually a date) in every query, so pruning skips most of the data.
- **Pre-aggregate** frequently used metrics into summary tables or materialized views rather than recomputing from raw events on every dashboard load.
- **Watch the query profile**: bytes scanned, partitions pruned, and spills to disk are the warehouse equivalents of `EXPLAIN ANALYZE`.

### Open table formats and the lakehouse

Increasingly, data lives as **Parquet** files in object storage (S3, GCS, Azure Blob), organized by an open **table format** (Apache Iceberg, Delta Lake) that adds transactions, schema evolution, and time travel on top of plain files. Many engines (Spark, Trino, DuckDB, Snowflake, BigQuery, Databricks) can query the same tables. The SQL is the same; the storage is shared.

**Notes:**
- Row stores for transactions, column stores for analytics. Most companies run both, with a pipeline in between.
- DuckDB is often the fastest way to analyze a few gigabytes of CSV or Parquet on a laptop — no cluster, no upload, plain SQL.
- `GROUP BY ALL`, `QUALIFY`, and `EXCLUDE` are not in PostgreSQL. When moving queries between engines, these are the first things to translate ([Section 54](#54-dialect-translation-table)).
- Warehouse cost scales with data scanned or compute time, so query design is also budget design.

---

## 50. SQL for Machine Learning and AI: Features, Splits, and Vector Search

Machine learning systems start and end in databases — training data is extracted with SQL, features are computed with SQL, predictions are written back to tables, and modern AI applications retrieve context for language models with vector search, increasingly inside PostgreSQL itself.

### Point-in-time feature engineering

A model predicts the future from the past, so every feature must be computed from data available **before** the moment of prediction. Using information from after that moment is **data leakage**: the model looks brilliant in evaluation and fails in production. The discipline is a **cutoff date**: features use only rows before it, and the label uses only rows after it.

Predicting whether a customer will order in the next 90 days, as of 2026-04-01, with classic **RFM** features (recency, frequency, monetary value):

```sql
WITH params AS (
    SELECT timestamptz '2026-04-01' AS cutoff, interval '90 days' AS horizon
),
history AS (                                    -- only what was known at the cutoff
    SELECT o.customer_id, o.order_id, o.ordered_at, oi.quantity * oi.unit_price AS amount
    FROM orders o JOIN order_items oi USING (order_id), params
    WHERE o.ordered_at < params.cutoff AND o.status <> 'cancelled'
),
features AS (
    SELECT c.customer_id,
           COALESCE(extract(day FROM (SELECT cutoff FROM params) - max(h.ordered_at))::int, 999)
                                                   AS recency_days,
           count(DISTINCT h.order_id)              AS frequency,
           COALESCE(sum(h.amount), 0)              AS monetary,
           (DATE '2026-04-01' - c.signup_date)     AS tenure_days
    FROM customers c
    LEFT JOIN history h USING (customer_id)
    GROUP BY c.customer_id, c.signup_date
),
labels AS (                                     -- only what happened after the cutoff
    SELECT DISTINCT o.customer_id, 1 AS ordered_next_90d
    FROM orders o, params
    WHERE o.ordered_at >= params.cutoff
      AND o.ordered_at <  params.cutoff + params.horizon
      AND o.status <> 'cancelled'
)
SELECT f.*, COALESCE(l.ordered_next_90d, 0) AS label
FROM features f
LEFT JOIN labels l USING (customer_id)
ORDER BY f.customer_id;
```

```text
 customer_id | recency_days | frequency | monetary | tenure_days | label
-------------+--------------+-----------+----------+-------------+-------
           1 |            1 |         3 |  1461.89 |         451 |     0
           2 |           78 |         1 |   127.00 |         438 |     1
           3 |           22 |         2 |  1197.98 |         423 |     1
           4 |           45 |         1 |   498.00 |         405 |     1
           5 |          999 |         0 |        0 |         386 |     1
           6 |           16 |         1 |   199.00 |         367 |     0
           7 |          999 |         0 |        0 |         352 |     1
           8 |          999 |         0 |        0 |         335 |     1
           9 |          999 |         0 |        0 |         314 |     0
          10 |          999 |         0 |        0 |         296 |     0
(10 rows)
```

Notice the defaults for customers with no history (`recency_days = 999`, `monetary = 0`): features must be defined for everyone the model will score, not only for active customers. For real training sets, repeat this for many cutoff dates (a monthly series of snapshots) to get more examples and to evaluate on later periods than you train on.

### Deterministic train/test splits

Splitting data randomly with `random()` gives a different split on every run, which makes experiments irreproducible. Hashing a stable ID gives the same assignment every time, on any engine, and keeps all rows of one entity on the same side of the split (which prevents another form of leakage):

```sql
SELECT customer_id,
       ('x' || left(md5(customer_id::text), 8))::bit(32)::bigint % 100 AS bucket,
       CASE WHEN ('x' || left(md5(customer_id::text), 8))::bit(32)::bigint % 100 < 80
            THEN 'train' ELSE 'test' END AS split
FROM customers
ORDER BY customer_id;
```

```text
 customer_id | bucket | split
-------------+--------+-------
           1 |     60 | train
           2 |      5 | train
           3 |     10 | train
           4 |     57 | train
           5 |     27 | train
           6 |     40 | train
           7 |     71 | train
           8 |     93 | test
           9 |      6 | train
          10 |      0 | train
(10 rows)
```

The expression takes the first 32 bits of an MD5 hash as a number and keeps its last two digits, a uniform bucket from 0 to 99. BigQuery users write `MOD(ABS(FARM_FINGERPRINT(CAST(id AS STRING))), 100)` — the idea is identical. With only ten customers the split is lumpy (9 to 1), while on thousands of rows it converges to 80/20.

### Sampling

Exploring or prototyping on a sample is faster than on the full table. `TABLESAMPLE` samples without scanning everything: `SYSTEM` picks random disk pages (fast, clumpy), `BERNOULLI` picks random rows (slower, uniform). `REPEATABLE (seed)` makes the sample reproducible:

```sql
CREATE TABLE big AS SELECT g AS id, g % 3 AS class FROM generate_series(1, 100000) AS g;

SELECT count(*) AS sampled_rows
FROM big TABLESAMPLE BERNOULLI (1) REPEATABLE (42);
```

```text
SELECT 100000
 sampled_rows
--------------
         1008
(1 row)
```

**Stratified sampling**, taking the same number (or share) from each class, which matters for imbalanced labels, is a window-function pattern:

```sql
SELECT class, count(*) AS rows
FROM (
    SELECT class, row_number() OVER (PARTITION BY class ORDER BY md5(id::text)) AS rn
    FROM big
) t
WHERE rn <= 5
GROUP BY class
ORDER BY class;
```

```text
 class | rows
-------+------
     0 |    5
     1 |    5
     2 |    5
(3 rows)
```

`ORDER BY random() LIMIT n` also samples, but sorts the entire table first, so it is only acceptable on small tables.

### Vector search with pgvector

An **embedding** is a list of numbers (a vector) produced by a model such that similar content gets nearby vectors. Semantic search, recommendations, deduplication, and **retrieval-augmented generation** (RAG, fetching relevant documents to give a language model as context) all reduce to one query — find the stored vectors nearest to a query vector.

The **pgvector** extension adds a `vector` type, distance operators, and approximate nearest-neighbor indexes to PostgreSQL, so embeddings can live in the same database as the data they describe, filtered and joined with ordinary SQL. Real embeddings have hundreds to thousands of dimensions; three dimensions keep this example readable:

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE doc_chunks (
    chunk_id   integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    product_id integer REFERENCES products,
    content    text NOT NULL,
    embedding  vector(3) NOT NULL          -- in production: vector(1536) or similar
);

INSERT INTO doc_chunks (product_id, content, embedding) VALUES
    (1,  'Laptop Pro 14: 16 GB RAM, great for developers',        '[0.90, 0.10, 0.05]'),
    (2,  'Laptop Air 13: light and thin, long battery life',       '[0.85, 0.20, 0.00]'),
    (7,  'Desktop Tower: 32 GB RAM workstation for heavy compute', '[0.70, 0.05, 0.30]'),
    (8,  'SQL for Everyone: learn queries from scratch',           '[0.05, 0.95, 0.10]'),
    (10, 'Designing Data Systems: databases at scale',             '[0.20, 0.85, 0.25]'),
    (11, 'Noise-Cancelling Headphones for focused work',           '[0.30, 0.10, 0.90]');
```

```text
CREATE EXTENSION
CREATE TABLE
INSERT 0 6
```

pgvector provides three distance operators. Smaller means more similar for all three:

| Operator | Distance | Use when |
|:--|:--|:--|
| `<->` | Euclidean (L2) | Embeddings are not normalized and magnitude matters |
| `<=>` | Cosine distance (1 minus cosine similarity) | The common default for text embeddings |
| `<#>` | Negative inner product | Embeddings are normalized (then it ranks like cosine, and is fastest) |

A **k-nearest-neighbor** query orders by distance to the query vector and takes the top k. Suppose a user asks "a book to learn databases" and the embedding model maps it to `[0.10, 0.90, 0.15]`:

```sql
SELECT chunk_id,
       content,
       round((embedding <=> '[0.10, 0.90, 0.15]')::numeric, 4) AS cosine_distance
FROM doc_chunks
ORDER BY embedding <=> '[0.10, 0.90, 0.15]'
LIMIT 3;
```

```text
 chunk_id |                     content                      | cosine_distance
----------+--------------------------------------------------+-----------------
        4 | SQL for Everyone: learn queries from scratch     |          0.0034
        5 | Designing Data Systems: databases at scale       |          0.0134
        2 | Laptop Air 13: light and thin, long battery life |          0.6694
(3 rows)
```

Because the vectors sit next to relational data, **filtered search** is a plain `WHERE` and join, for example only products that are in stock and under a price limit:

```sql
SELECT p.name, p.price, p.stock,
       round((c.embedding <=> '[0.80, 0.15, 0.10]')::numeric, 4) AS distance
FROM doc_chunks c
JOIN products p USING (product_id)
WHERE p.stock > 0 AND p.price < 1400
ORDER BY c.embedding <=> '[0.80, 0.15, 0.10]'
LIMIT 2;
```

```text
     name      |  price  | stock | distance
---------------+---------+-------+----------
 Laptop Pro 14 | 1299.00 |    15 |   0.0050
 Laptop Air 13 |  999.00 |    20 |   0.0085
(2 rows)
```

Exact search compares the query with every row, which is fine up to perhaps a hundred thousand vectors. Beyond that, create an **approximate nearest neighbor** index. **HNSW** (a navigable graph) gives the best speed and recall trade-off; **IVFFlat** (clustered lists) builds faster and uses less memory. The index must match the operator used in queries:

```sql
CREATE INDEX idx_chunks_embedding ON doc_chunks USING hnsw (embedding vector_cosine_ops);
```

```text
CREATE INDEX
```

Approximate indexes trade a little recall for a lot of speed, and tuning parameters (`m` and `ef_construction` at build time, `hnsw.ef_search` at query time) move along that trade-off.

### Hybrid search

Vector search captures meaning but can miss exact terms (product codes, names); keyword search ([Section 40](#40-json-arrays-and-full-text-search)) catches exact terms but misses paraphrases. **Hybrid search** runs both and merges the rankings. **Reciprocal rank fusion** (RRF) is a simple, robust merge: each result scores `1 / (k + rank)` in each list, summed across lists:

```sql
WITH semantic AS (
    SELECT chunk_id, row_number() OVER (ORDER BY embedding <=> '[0.60, 0.40, 0.20]') AS rnk
    FROM doc_chunks
    ORDER BY embedding <=> '[0.60, 0.40, 0.20]'
    LIMIT 5
),
keyword AS (
    SELECT chunk_id,
           row_number() OVER (ORDER BY ts_rank(to_tsvector('english', content), q) DESC) AS rnk
    FROM doc_chunks, websearch_to_tsquery('english', 'RAM') AS q
    WHERE to_tsvector('english', content) @@ q
)
SELECT d.chunk_id,
       d.content,
       round(COALESCE(1.0 / (60 + s.rnk), 0) + COALESCE(1.0 / (60 + k.rnk), 0), 5) AS rrf_score
FROM doc_chunks d
LEFT JOIN semantic s USING (chunk_id)
LEFT JOIN keyword  k USING (chunk_id)
WHERE s.chunk_id IS NOT NULL OR k.chunk_id IS NOT NULL
ORDER BY rrf_score DESC
LIMIT 3;
```

```text
 chunk_id |                        content                         | rrf_score
----------+--------------------------------------------------------+-----------
        1 | Laptop Pro 14: 16 GB RAM, great for developers         |   0.03227
        3 | Desktop Tower: 32 GB RAM workstation for heavy compute |   0.03226
        2 | Laptop Air 13: light and thin, long battery life       |   0.01639
(3 rows)
```

The two chunks mentioning RAM rise to the top because both rankers agree on them.

### The RAG retrieval loop

A retrieval-augmented generation system is mostly SQL around one model call:

1. **Ingest:** split documents into chunks, compute an embedding per chunk with an embedding model, and `INSERT` chunk text, metadata, and embedding (in batches, or with `COPY`).
2. **Retrieve:** embed the user's question with the same model, run a filtered kNN or hybrid query (restricted by tenant, permissions, and freshness with ordinary `WHERE` clauses and row-level security), and take the top k chunks.
3. **Generate:** send the question plus the retrieved chunks to the language model.
4. **Log:** store the question, retrieved chunk IDs, answer, latency, and feedback in a table (a `jsonb` column suits the variable parts) for evaluation and debugging.

Keeping embeddings in PostgreSQL means one system handles transactions, permissions, joins, backups, and vectors. Dedicated vector databases (Pinecone, Qdrant, Weaviate, Milvus) become worthwhile at very large scale or for specialized features.

### Text-to-SQL and database agents

Language models are good at writing SQL, and AI engineers increasingly build assistants that answer questions by generating and running queries. The engineering that makes them safe and accurate is database engineering, not prompting:

- **Least privilege:** connect as a role with `SELECT` only on approved tables or views, `default_transaction_read_only`, and a `statement_timeout` ([Section 44](#44-roles-permissions-and-row-level-security)).
- **Curated surface:** expose clean views with clear column names and comments (`COMMENT ON COLUMN ...`), not raw normalized tables — the model writes better SQL against a well-named star schema.
- **Schema context:** give the model table definitions, relationships, sample values, and definitions of business metrics, the same documentation a new analyst would need.
- **Guardrails in code:** parse or `EXPLAIN` the generated SQL before running it, reject anything that is not a single `SELECT`, add a `LIMIT`, and log every query.
- **Evaluation:** keep a set of questions with known correct SQL results and measure accuracy on every change of prompt, model, or schema.

**Notes:**
- Leakage is the most expensive ML bug and SQL is where it happens — always compute features strictly before a cutoff and labels strictly after it.
- Split by hashing a stable entity ID, not by `random()`, so experiments are reproducible and entities never straddle train and test.
- Use the same embedding model for stored documents and queries, and the index operator class that matches your distance operator (`vector_cosine_ops` for `<=>`).
- Combine vector similarity with ordinary SQL filters and joins — that combination is pgvector's main advantage over a separate vector store.
- Other engines have vector search too (DuckDB's `vss` extension, BigQuery `VECTOR_SEARCH`, Snowflake `VECTOR` type, SQL Server 2025 vectors), with different syntax and the same concepts.

---

## 51. Common Mistakes

**Comparing with `= NULL`.** `x = NULL` is never true, so the filter silently matches nothing. Use `IS NULL`, `IS NOT NULL`, or `IS DISTINCT FROM` ([Section 8](#8-null-and-three-valued-logic)).

**`NOT IN` with a nullable subquery.** One `NULL` in the list makes `NOT IN` return no rows at all — use `NOT EXISTS` for anti joins.

**Forgetting that `<>` drops `NULL` rows.** `WHERE city <> 'London'` also excludes rows with no city. Add `OR city IS NULL` when unknowns should be included.

**Mixing `AND` and `OR` without parentheses.** `AND` binds tighter, so `a OR b AND c` means `a OR (b AND c)`. Parenthesize every mixed condition.

**Using a `SELECT` alias in `WHERE`.** `WHERE` runs before `SELECT`, so the alias does not exist yet. Repeat the expression or wrap the query in a CTE.

**Filtering aggregates in `WHERE`.** Aggregate conditions belong in `HAVING`, which runs after grouping.

**Selecting ungrouped columns.** Every selected column must be grouped or aggregated. MySQL's lenient mode silently returns an arbitrary value instead of an error.

**Integer division.** `3 / 15` is `0` — multiply by `100.0` or cast to `numeric` before dividing.

**Dividing by a value that can be zero.** Wrap the denominator in `NULLIF(x, 0)`.

**Storing money in floating point.** `0.1 + 0.2` is not `0.3` in `double precision`. Use `numeric`.

**`BETWEEN` on timestamps.** `BETWEEN '2026-01-01' AND '2026-01-31'` misses everything after midnight on the 31st. Use `>= start AND < next_start`.

**Functions on indexed columns in `WHERE`.** `WHERE created_at::date = '2026-03-01'` or `WHERE lower(email) = ...` cannot use a plain index on the column. Rewrite as a range or add an expression index.

**The join fan-out.** Joining a one-to-many relationship and then summing or counting values from the "one" side inflates totals. Know each table's grain, and aggregate before joining.

**`WHERE` on the right table of a `LEFT JOIN`.** It removes the `NULL`-extended rows and turns the left join into an inner join. Put conditions on the optional table in `ON`.

**`count(*)` after a `LEFT JOIN`.** It counts the `NULL`-extended row as 1. Count a column from the right table instead.

**`LIMIT` without `ORDER BY`.** Returns arbitrary rows that can change between runs. Always sort before limiting, with a unique tiebreaker for stable pagination.

**Deep `OFFSET` pagination.** `OFFSET 100000` still reads 100,000 rows. Use keyset pagination (`WHERE id > last_seen ORDER BY id LIMIT n`).

**`UNION` where `UNION ALL` was meant.** `UNION` removes duplicates — costing a sort, and sometimes deleting legitimate repeated rows.

**`UPDATE` or `DELETE` without `WHERE`.** Changes every row. Run the `WHERE` as a `SELECT` first and use a transaction for risky changes.

**Building SQL with string formatting.** f-strings and concatenation invite SQL injection. Pass values as parameters; allow-list identifiers.

**Forgetting foreign key indexes.** Primary keys are indexed automatically; foreign keys are not. Unindexed foreign keys make joins and parent deletes slow.

**The N+1 query pattern.** One query per row from an application loop multiplies round trips. Use a join, `IN (...)`, or eager loading in the ORM.

**Read-modify-write in application code.** Two sessions reading and writing the same value lose updates. Use atomic `UPDATE ... SET x = x - 1 WHERE x >= 1` or row locks.

**Long-open transactions.** An idle session inside `BEGIN` holds locks and blocks vacuum — keep transactions short and always end them.

**Leaking the future into ML features.** Computing features over all data, including rows after the prediction time, produces models that fail in production. Always use a cutoff.

**Trusting warehouse constraints.** Most warehouses do not enforce primary or foreign keys — test uniqueness and relationships explicitly.

---

## 52. Interview Quick-Fire

Short, high-frequency questions with the answers an interviewer expects, followed by the classic hands-on problems.

**What is the difference between `WHERE` and `HAVING`?** `WHERE` filters rows before grouping and cannot use aggregates; `HAVING` filters groups after aggregation and can.

**What is the logical order of a `SELECT`?** `FROM`/`JOIN`, `WHERE`, `GROUP BY`, `HAVING`, `SELECT` (including window functions), `DISTINCT`, `ORDER BY`, `LIMIT`.

**What does `NULL = NULL` return?** `NULL` (unknown), which `WHERE` treats as not true. Use `IS NULL`.

**`count(*)` vs `count(col)` vs `count(DISTINCT col)`?** All rows; rows where `col` is not `NULL`; distinct non-null values.

**Inner vs left vs full join?** Inner keeps matched pairs only; left keeps every left row, with `NULL`s where unmatched; full keeps unmatched rows from both sides.

**`UNION` vs `UNION ALL`?** `UNION` removes duplicate rows (with a sort or hash); `UNION ALL` keeps everything and is faster.

**`ROW_NUMBER` vs `RANK` vs `DENSE_RANK`?** For 100, 90, 90, 80: 1-2-3-4, 1-2-2-4, 1-2-2-3.

**`DELETE` vs `TRUNCATE` vs `DROP`?** `DELETE` removes selected rows, row by row, with triggers; `TRUNCATE` empties the table instantly; `DROP` removes the table itself.

**Primary key vs unique key?** Both enforce uniqueness. A table has one primary key, which cannot be `NULL`; it can have many unique keys, which allow `NULL`s by default.

**What is a foreign key?** A column whose values must exist in another table's key, enforcing referential integrity.

**What is normalization?** Organizing tables so each fact is stored once, avoiding update, insert, and delete anomalies. 1NF: atomic values. 2NF: no partial dependency on a composite key. 3NF: no dependency between non-key columns.

**When would you denormalize?** For read-heavy analytics — star schemas, materialized views — where fewer joins matter more than write simplicity.

**What is an index, and what does it cost?** A sorted structure (usually a B-tree) that finds rows without a full scan. It costs disk space — and it slows every write.

**Clustered vs non-clustered index?** A clustered index defines the physical order of the table's rows (SQL Server and MySQL InnoDB primary keys); non-clustered indexes are separate structures pointing to rows. PostgreSQL tables are heaps — every index is non-clustered.

**Why might a query not use an index?** A non-sargable predicate, low selectivity, stale statistics, a type mismatch, or a composite index whose leading column is not filtered.

**What is ACID?** Atomicity, Consistency, Isolation, Durability: the guarantees of a transaction.

**Name the isolation levels.** Read Uncommitted, Read Committed, Repeatable Read, Serializable, each preventing more anomalies (dirty reads, non-repeatable reads, phantoms, write skew).

**What is a CTE, and when is one recursive?** A named subquery defined with `WITH`. It is recursive when it references itself, which walks hierarchies and graphs.

**Correlated vs non-correlated subquery?** A correlated subquery references the outer row and conceptually runs per row; a non-correlated one runs once.

**`EXISTS` vs `IN`?** Equivalent for positive membership. For negation, `NOT EXISTS` is `NULL`-safe while `NOT IN` breaks on `NULL`s.

**What is a view vs a materialized view?** A view is a stored query run on every read; a materialized view stores its result and must be refreshed.

**OLTP vs OLAP?** OLTP: many small transactions on current data, row-oriented, normalized. OLAP: large analytical scans over history, column-oriented, denormalized.

**Star vs snowflake schema?** Both center on a fact table; a star's dimensions are single denormalized tables, a snowflake's are normalized into sub-dimensions.

**SCD Type 1 vs Type 2?** Type 1 overwrites the attribute; Type 2 closes the old row and inserts a new version with validity dates, preserving history.

**What is SQL injection and how do you prevent it?** Untrusted input changing a query's meaning when pasted into SQL text. Prevent it with parameterized queries and allow-listed identifiers.

**What is the N+1 problem?** One query for a list, plus one per item — fix it with joins, `IN` lists, or eager loading.

### Classic problems

**Second highest salary (handling ties):**

```sql
SELECT max(salary) AS second_highest
FROM employees
WHERE salary < (SELECT max(salary) FROM employees);
```

```text
 second_highest
----------------
      150000.00
(1 row)
```

**Nth highest, generalized (N = 3):**

```sql
SELECT DISTINCT salary
FROM (SELECT salary, dense_rank() OVER (ORDER BY salary DESC) AS dr FROM employees) t
WHERE dr = 3;
```

```text
  salary
-----------
 120000.00
(1 row)
```

**Duplicate values:**

```sql
SELECT salary, count(*) AS people
FROM employees
GROUP BY salary
HAVING count(*) > 1;
```

```text
  salary  | people
----------+--------
 60000.00 |      2
 95000.00 |      2
(2 rows)
```

**Rows with no match (customers who never ordered):**

```sql
SELECT c.first_name
FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);
```

```text
 first_name
------------
 Jonas
 Isla
(2 rows)
```

**Highest earner per department (top 1 per group):**

```sql
SELECT department, first_name, salary
FROM (
    SELECT *, rank() OVER (PARTITION BY department ORDER BY salary DESC) AS r
    FROM employees
) t
WHERE r = 1
ORDER BY department;
```

```text
 department  | first_name |  salary
-------------+------------+-----------
 Engineering | Marcus     | 150000.00
 Executive   | Sofia      | 180000.00
 Sales       | Priya      | 120000.00
 Support     | Eva        |  70000.00
(4 rows)
```

**Employees earning above their department average:**

```sql
SELECT first_name, department, salary
FROM (SELECT *, avg(salary) OVER (PARTITION BY department) AS dept_avg FROM employees) t
WHERE salary > dept_avg
ORDER BY department;
```

```text
 first_name | department  |  salary
------------+-------------+-----------
 Marcus     | Engineering | 150000.00
 Priya      | Sales       | 120000.00
(2 rows)
```

**Running total per customer:**

```sql
SELECT customer_id, order_id,
       count(*) OVER (PARTITION BY customer_id ORDER BY ordered_at ROWS UNBOUNDED PRECEDING) AS orders_so_far
FROM orders
WHERE customer_id = 3;
```

```text
 customer_id | order_id | orders_so_far
-------------+----------+---------------
           3 |        4 |             1
           3 |        7 |             2
           3 |       15 |             3
(3 rows)
```

**Customers who ordered in two consecutive months:**

```sql
WITH months AS (
    SELECT DISTINCT customer_id, date_trunc('month', ordered_at)::date AS m
    FROM orders WHERE status <> 'cancelled'
)
SELECT DISTINCT a.customer_id
FROM months a
JOIN months b ON b.customer_id = a.customer_id AND b.m = a.m + interval '1 month';
```

```text
 customer_id
-------------
           3
(1 row)
```

**Share of each category in total revenue:**

```sql
SELECT cat.name,
       round(100.0 * sum(oi.quantity * oi.unit_price) / sum(sum(oi.quantity * oi.unit_price)) OVER (), 1) AS pct
FROM order_items oi
JOIN products p USING (product_id)
JOIN categories cat USING (category_id)
GROUP BY cat.name
ORDER BY pct DESC;
```

```text
    name     | pct
-------------+------
 Laptops     | 70.7
 Accessories | 13.9
 Computers   |  9.1
 Programming |  6.3
(4 rows)
```

---

## 53. Cheat Sheet

**Query skeleton (in logical order: FROM, WHERE, GROUP BY, HAVING, SELECT, DISTINCT, ORDER BY, LIMIT)**

```sql
WITH cte AS (SELECT ...)                          -- optional named steps
SELECT DISTINCT a.col, agg(b.col) AS alias,       -- columns and expressions
       fn(...) OVER (PARTITION BY ... ORDER BY ... ROWS BETWEEN ... AND ...) AS w
FROM table_a AS a
[INNER | LEFT | RIGHT | FULL | CROSS] JOIN table_b AS b ON b.key = a.key
WHERE row_condition
GROUP BY a.col
HAVING group_condition
ORDER BY alias DESC NULLS LAST
LIMIT n OFFSET m;
```

**Filtering**

| Task | SQL |
|:--|:--|
| Equality, inequality | `=`, `<>` |
| Range (inclusive) | `x BETWEEN a AND b` |
| Time range (half-open) | `ts >= '2026-01-01' AND ts < '2026-02-01'` |
| Membership | `x IN (1, 2, 3)` |
| Pattern | `LIKE 'a%'`, `ILIKE '%a%'`, `~ '^a\d+'` |
| Missing values | `IS NULL`, `IS NOT NULL`, `IS DISTINCT FROM` |
| Has / lacks a match | `EXISTS (subquery)`, `NOT EXISTS (subquery)` |

**Aggregates and windows**

| Task | SQL |
|:--|:--|
| Count rows, values, distinct values | `count(*)`, `count(col)`, `count(DISTINCT col)` |
| Conditional aggregate | `count(*) FILTER (WHERE cond)`, `sum(CASE WHEN cond THEN x END)` |
| Median, percentile | `percentile_cont(0.5) WITHIN GROUP (ORDER BY x)` |
| Concatenate values | `string_agg(x, ', ' ORDER BY x)` |
| Subtotals | `GROUP BY ROLLUP (a, b)`, `CUBE`, `GROUPING SETS` |
| Rank | `row_number()`, `rank()`, `dense_rank()`, `ntile(n)` |
| Neighbor rows | `lag(x)`, `lead(x)`, `first_value(x)` |
| Running total | `sum(x) OVER (ORDER BY t ROWS UNBOUNDED PRECEDING)` |
| Moving average | `avg(x) OVER (ORDER BY t ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)` |
| Share of total | `x / sum(x) OVER ()` |

**Functions**

| Task | SQL |
|:--|:--|
| Default for NULL | `COALESCE(x, default)` |
| Safe division | `a / NULLIF(b, 0)` |
| Cast | `CAST(x AS integer)`, `x::integer` |
| Round | `round(x, 2)` |
| Text | `lower`, `upper`, `trim`, `length`, `substring`, `split_part`, `replace`, `concat_ws` |
| Truncate date | `date_trunc('month', ts)` |
| Extract part | `extract(year FROM ts)` |
| Date math | `d + 7`, `ts + interval '1 hour'`, `d2 - d1`, `age(d2, d1)` |
| Calendar spine | `generate_series(start, stop, interval '1 day')` |
| JSON value | `j ->> 'key'`, `j @> '{"k": "v"}'` |

**Changing data and structure**

```sql
INSERT INTO t (a, b) VALUES (1, 'x'), (2, 'y') RETURNING id;
INSERT INTO t (a, b) SELECT a, b FROM staging;
INSERT INTO t (k, v) VALUES (1, 'x') ON CONFLICT (k) DO UPDATE SET v = EXCLUDED.v;
UPDATE t SET a = a + 1 WHERE id = 5;
UPDATE t SET a = s.a FROM source s WHERE s.id = t.id;
DELETE FROM t WHERE id = 5;
MERGE INTO t USING s ON s.id = t.id
  WHEN MATCHED THEN UPDATE SET a = s.a
  WHEN NOT MATCHED THEN INSERT (id, a) VALUES (s.id, s.a);

CREATE TABLE t (id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
                name text NOT NULL UNIQUE,
                parent_id bigint REFERENCES t (id) ON DELETE CASCADE,
                price numeric(10,2) CHECK (price > 0),
                created_at timestamptz NOT NULL DEFAULT now());
ALTER TABLE t ADD COLUMN note text;
CREATE INDEX idx_t_parent ON t (parent_id);
CREATE VIEW v AS SELECT ...;
CREATE MATERIALIZED VIEW mv AS SELECT ...;  REFRESH MATERIALIZED VIEW mv;

BEGIN; ... COMMIT;      -- or ROLLBACK;
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
```

**psql essentials**

| Command | Does |
|:--|:--|
| `\l`, `\c db` | List databases, connect |
| `\dt`, `\d table` | List tables, describe one |
| `\x`, `\timing` | Expanded output, show timings |
| `\copy t FROM 'f.csv' CSV HEADER` | Load a local CSV |
| `\i file.sql`, `\q` | Run a script, quit |

---

## 54. Dialect Translation Table

The same task in the major transactional databases. PostgreSQL syntax is what this guide uses.

| Task | PostgreSQL | MySQL | SQLite | SQL Server | Oracle |
|:--|:--|:--|:--|:--|:--|
| First n rows | `LIMIT n` | `LIMIT n` | `LIMIT n` | `TOP (n)` / `OFFSET 0 ROWS FETCH NEXT n ROWS ONLY` | `FETCH FIRST n ROWS ONLY` |
| Auto-increment key | `GENERATED ALWAYS AS IDENTITY` | `AUTO_INCREMENT` | `INTEGER PRIMARY KEY` | `IDENTITY(1,1)` | `GENERATED AS IDENTITY` |
| Concatenate | `a \|\| b` | `CONCAT(a, b)` | `a \|\| b` | `a + b` / `CONCAT(a, b)` | `a \|\| b` |
| Null default | `COALESCE` | `IFNULL` / `COALESCE` | `IFNULL` / `COALESCE` | `ISNULL` / `COALESCE` | `NVL` / `COALESCE` |
| Current timestamp | `now()` | `NOW()` | `datetime('now')` | `SYSDATETIME()` | `SYSTIMESTAMP` |
| Truncate to month | `date_trunc('month', d)` | `DATE_FORMAT(d, '%Y-%m-01')` | `date(d, 'start of month')` | `DATETRUNC(month, d)` (2022) | `TRUNC(d, 'MM')` |
| Add 7 days | `d + interval '7 days'` | `DATE_ADD(d, INTERVAL 7 DAY)` | `date(d, '+7 days')` | `DATEADD(day, 7, d)` | `d + 7` |
| Days between | `d2 - d1` | `DATEDIFF(d2, d1)` | `julianday(d2) - julianday(d1)` | `DATEDIFF(day, d1, d2)` | `d2 - d1` |
| String length | `length(s)` | `CHAR_LENGTH(s)` | `length(s)` | `LEN(s)` | `LENGTH(s)` |
| Find substring | `strpos(s, 'x')` | `LOCATE('x', s)` | `instr(s, 'x')` | `CHARINDEX('x', s)` | `INSTR(s, 'x')` |
| Case-insensitive match | `ILIKE` | `LIKE` (default collation) | `LIKE` (ASCII) | `LIKE` (default collation) | `UPPER(a) LIKE UPPER(b)` |
| Regex match | `s ~ 'p'` | `s REGEXP 'p'` | extension needed | `REGEXP_LIKE` (2025) | `REGEXP_LIKE(s, 'p')` |
| Aggregate to string | `string_agg(x, ',')` | `GROUP_CONCAT(x)` | `group_concat(x, ',')` | `STRING_AGG(x, ',')` | `LISTAGG(x, ',')` |
| Upsert | `ON CONFLICT ... DO UPDATE` | `ON DUPLICATE KEY UPDATE` | `ON CONFLICT ... DO UPDATE` | `MERGE` | `MERGE` |
| Return written rows | `RETURNING` | not available | `RETURNING` | `OUTPUT` | `RETURNING INTO` |
| Boolean type | `boolean` | `BOOLEAN` (a `TINYINT(1)`) | integer 0/1 | `BIT` | `BOOLEAN` (recent versions) |
| Quote identifier | `"name"` | `` `name` `` | `"name"` | `[name]` | `"name"` |
| Full outer join | yes | emulate with `UNION` | yes (3.39+) | yes | yes |
| Set difference | `EXCEPT` | `EXCEPT` (8.0.31+) | `EXCEPT` | `EXCEPT` | `MINUS` / `EXCEPT` |
| Number series | `generate_series` | recursive CTE | recursive CTE | `GENERATE_SERIES` (2022) | `CONNECT BY LEVEL` |
| Recursive CTE | `WITH RECURSIVE` | `WITH RECURSIVE` | `WITH RECURSIVE` | `WITH` | `WITH` |
| JSON field as text | `j ->> 'k'` | `j ->> '$.k'` | `j ->> '$.k'` | `JSON_VALUE(j, '$.k')` | `JSON_VALUE(j, '$.k')` |
| Procedural language | PL/pgSQL | SQL/PSM style | none | T-SQL | PL/SQL |

And in analytical engines, compared with PostgreSQL:

| Task | PostgreSQL | DuckDB | BigQuery | Snowflake | Databricks (Spark SQL) |
|:--|:--|:--|:--|:--|:--|
| Filter on window result | subquery or CTE | `QUALIFY` | `QUALIFY` | `QUALIFY` | `QUALIFY` |
| Group by every selected column | list them | `GROUP BY ALL` | `GROUP BY ALL` | `GROUP BY ALL` | `GROUP BY ALL` |
| All columns except some | list them | `* EXCLUDE (a)` | `* EXCEPT (a)` | `* EXCLUDE (a)` | `* EXCEPT (a)` |
| Truncate to month | `date_trunc('month', d)` | `date_trunc('month', d)` | `DATE_TRUNC(d, MONTH)` | `DATE_TRUNC('month', d)` | `date_trunc('month', d)` |
| Cast returning NULL on failure | not built in | `TRY_CAST` | `SAFE_CAST` | `TRY_CAST` | `try_cast` |
| Array to rows | `unnest(a)` | `unnest(a)` | `UNNEST(a)` in `FROM` | `LATERAL FLATTEN(a)` | `explode(a)` |
| JSON field | `j ->> 'k'` | `j ->> 'k'` | `JSON_VALUE(j, '$.k')` | `j:k::string` | `j:k` |
| Approximate distinct count | extension | `approx_count_distinct` | `APPROX_COUNT_DISTINCT` | `APPROX_COUNT_DISTINCT` | `approx_count_distinct` |
| Quote identifier | `"name"` | `"name"` | `` `name` `` | `"name"` | `` `name` `` |

---

## 55. Keyword Reference

SQL keywords are case-insensitive. The most important ones, grouped by purpose, with the sections that cover them.

**Querying**

| Keyword | What it does | Covered in |
|:--|:--|:--|
| `SELECT` | Chooses output columns and expressions | 6 |
| `FROM` | Names the source tables | 6 |
| `AS` | Aliases a column or table | 6 |
| `WHERE` | Filters rows before grouping | 7 |
| `AND`, `OR`, `NOT` | Combine conditions with three-valued logic | 7, 8 |
| `IN`, `BETWEEN`, `LIKE`, `ILIKE` | Membership, inclusive range, pattern tests | 7 |
| `IS [NOT] NULL`, `IS [NOT] DISTINCT FROM` | Null tests and null-safe comparison | 8 |
| `ORDER BY`, `ASC`, `DESC`, `NULLS FIRST` | Sort the result | 9 |
| `LIMIT`, `OFFSET`, `FETCH FIRST ... WITH TIES` | Restrict the number of rows | 9 |
| `DISTINCT`, `DISTINCT ON` | Remove duplicates; first row per group | 9 |
| `CASE ... WHEN ... THEN ... ELSE ... END` | Conditional expression | 10 |
| `CAST`, `::` | Type conversion | 11 |

**Aggregation**

| Keyword | What it does | Covered in |
|:--|:--|:--|
| `GROUP BY` | Collapse rows into groups | 17 |
| `HAVING` | Filter groups after aggregation | 17 |
| `FILTER (WHERE ...)` | Restrict rows fed to one aggregate | 18 |
| `ROLLUP`, `CUBE`, `GROUPING SETS`, `GROUPING()` | Subtotals and grand totals | 18 |
| `WITHIN GROUP (ORDER BY ...)` | Ordered-set aggregates such as percentiles | 16 |

**Combining**

| Keyword | What it does | Covered in |
|:--|:--|:--|
| `JOIN`, `INNER JOIN`, `ON`, `USING` | Combine matching rows | 20 |
| `LEFT`, `RIGHT`, `FULL [OUTER] JOIN` | Keep unmatched rows | 21 |
| `CROSS JOIN`, `LATERAL` | Cartesian product; per-row subquery | 22 |
| `EXISTS`, `ANY`, `ALL` | Subquery predicates | 22, 24 |
| `UNION [ALL]`, `INTERSECT`, `EXCEPT` | Set operations | 23 |
| `WITH`, `WITH RECURSIVE` | Common table expressions | 25, 26 |

**Windows**

| Keyword | What it does | Covered in |
|:--|:--|:--|
| `OVER`, `PARTITION BY`, `WINDOW` | Define a window | 27 |
| `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE` | Ranking functions | 28 |
| `LAG`, `LEAD`, `FIRST_VALUE`, `LAST_VALUE` | Offset and value functions | 28 |
| `ROWS`, `RANGE`, `GROUPS`, `UNBOUNDED PRECEDING`, `CURRENT ROW` | Window frames | 29 |

**Changing data**

| Keyword | What it does | Covered in |
|:--|:--|:--|
| `INSERT INTO ... VALUES`, `DEFAULT` | Add rows | 32 |
| `RETURNING` | Return written rows | 32 |
| `UPDATE ... SET`, `UPDATE ... FROM` | Change rows | 33 |
| `DELETE`, `DELETE ... USING`, `TRUNCATE` | Remove rows | 33 |
| `ON CONFLICT`, `EXCLUDED`, `MERGE` | Upserts and synchronization | 34 |

**Defining structure**

| Keyword | What it does | Covered in |
|:--|:--|:--|
| `CREATE`, `ALTER`, `DROP` `TABLE` / `SCHEMA` | Manage tables and namespaces | 35 |
| `GENERATED ... AS IDENTITY`, `GENERATED ALWAYS AS (...) STORED` | Identity and generated columns | 35 |
| `PRIMARY KEY`, `FOREIGN KEY`, `REFERENCES` | Keys and relationships | 19, 36 |
| `NOT NULL`, `UNIQUE`, `CHECK`, `DEFAULT`, `CONSTRAINT` | Constraints | 36 |
| `ON DELETE CASCADE / SET NULL / RESTRICT` | Foreign key actions | 36 |
| `CREATE [MATERIALIZED] VIEW`, `REFRESH` | Views | 38 |
| `CREATE INDEX`, `UNIQUE`, `INCLUDE`, `USING gin` | Indexes | 39 |
| `PARTITION BY RANGE`, `PARTITION OF` | Declarative partitioning | 48 |

**Transactions, security, and programmability**

| Keyword | What it does | Covered in |
|:--|:--|:--|
| `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` | Transaction control | 41 |
| `ISOLATION LEVEL`, `FOR UPDATE`, `SKIP LOCKED` | Isolation and row locking | 41 |
| `EXPLAIN`, `ANALYZE`, `VACUUM` | Plans and statistics | 42 |
| `CREATE FUNCTION`, `CREATE PROCEDURE`, `CALL`, `DO` | Server-side code | 43 |
| `CREATE TRIGGER`, `NEW`, `OLD` | Automatic row-level actions | 43 |
| `CREATE ROLE`, `GRANT`, `REVOKE`, `SET ROLE` | Access control | 44 |
| `CREATE POLICY`, `ENABLE ROW LEVEL SECURITY` | Row-level security | 44 |
| `COPY`, `\copy` | Bulk import and export | 45 |
| `CREATE EXTENSION` | Add features such as `vector` and `pg_trgm` | 40, 50 |
| `TABLESAMPLE` | Random sampling | 50 |
