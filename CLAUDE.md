# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A personal SQL learning repo. The user is learning MySQL from scratch, progressing through topics in notebook order. Claude acts as tutor — explaining concepts, answering questions about queries, and generating exams.

## Database

**Sakila** — MySQL's official DVD rental sample database. Loaded into a Docker container.

```bash
# Start container (credentials: root / practice)
docker start sql-practice

# Load schema (CLI only — DELIMITER syntax does not work via JDBC/notebooks)
docker exec -i sql-practice mysql -u root -ppractice < schema/setup.sql

# Interactive shell
docker exec -it sql-practice mysql -u root -ppractice sakila
```

**Connection string used in all notebooks:**
```
mysql+pymysql://root:practice@127.0.0.1:3306/sakila
```

## Sakila Schema

```
                   ┌──────────┐
                   │ language │◄──────────────┐
                   └────┬─────┘               │
                        │                     │
┌────────┐   ┌──────────▼──────────────────────────┐   ┌──────────┐
│ actor  │   │               film                  │   │ category │
│────────│   │─────────────────────────────────────│   │──────────│
│actor_id│   │film_id  title  rating  rental_rate  │   │category_id│
└───┬────┘   │rental_duration  length  language_id │   └────┬─────┘
    │        └──────┬──────────────────────┬───────┘        │
    │               │                      │                │
    │        ┌──────▼──────┐    ┌──────────▼──────┐        │
    └───────►│ film_actor  │    │  film_category  │◄───────┘
             │─────────────│    │─────────────────│
             │actor_id(FK) │    │film_id(FK)      │
             │film_id(FK)  │    │category_id(FK)  │
             └──────┬──────┘    └─────────────────┘
                    │
             ┌──────▼──────────┐      ┌──────────────┐
             │    inventory    │      │    store     │
             │─────────────────│      │──────────────│
             │inventory_id     │      │store_id      │◄──┐
             │film_id(FK)      │      │address_id(FK)│   │
             │store_id(FK)─────┼─────►│manager_staff │   │
             └──────┬──────────┘      └──────────────┘   │
                    │                                     │
             ┌──────▼──────────┐      ┌──────────────┐   │
             │     rental      │      │    staff     │   │
             │─────────────────│      │──────────────│   │
             │rental_id        │      │staff_id      ├───┘
             │inventory_id(FK) │      │address_id(FK)│
             │customer_id(FK)──┼──┐   │store_id(FK)  │
             │staff_id(FK)─────┼──┼──►│email  active │
             └──────┬──────────┘  │   └──────────────┘
                    │             │
             ┌──────▼──────────┐  │   ┌──────────────┐
             │     payment     │  │   │   customer   │
             │─────────────────│  │   │──────────────│
             │payment_id       │  └──►│customer_id   │
             │rental_id(FK)    │      │address_id(FK)│
             │customer_id(FK)  │      │store_id(FK)  │
             │staff_id(FK)     │      │email  active │
             │amount           │      └──────┬───────┘
             └─────────────────┘             │
                                      ┌──────▼───────┐
                                      │   address    │
                                      │──────────────│
                                      │address_id    │
                                      │city_id(FK)───┼──► city ──► country
                                      │phone         │
                                      └──────────────┘
```

| Table           | Rows   | Purpose                                   |
|-----------------|--------|-------------------------------------------|
| `film`          | 1,000  | Movie catalog                             |
| `actor`         | 200    | Actors                                    |
| `film_actor`    | 5,462  | Many-to-many: films ↔ actors              |
| `film_category` | 1,000  | Many-to-many: films ↔ categories          |
| `category`      | 16     | Genre categories                          |
| `language`      | 6      | Film languages                            |
| `inventory`     | 4,581  | Physical copies of films per store        |
| `rental`        | 16,044 | Each rental transaction                   |
| `payment`       | 16,049 | Payments per rental                       |
| `customer`      | 599    | Store customers                           |
| `staff`         | 2      | Store employees                           |
| `store`         | 2      | Physical store locations                  |
| `address`       | 603    | Addresses for customers/staff/stores      |
| `city`          | 600    | Cities                                    |
| `country`       | 109    | Countries                                 |

## Notebook Structure

One `.ipynb` file per topic in the root directory. Each notebook follows the same pattern:
- Setup cells: `%pip install jupysql pymysql` then `%load_ext sql` + `%sql <connection>`
- Concept sections: markdown cell explaining the concept with syntax examples
- Practice cells: `%%sql` magic with a question as a comment — **the user writes the SQL, cells are intentionally empty**

| Notebook | Topic |
|---|---|
| `01-select-basics.ipynb` | SELECT, aliases, computed columns, DISTINCT, ORDER BY, LIMIT/OFFSET |
| `02-filtering.ipynb` | WHERE, AND/OR/NOT, IN, BETWEEN, LIKE, IS NULL |
| `03-string-functions.ipynb` | CONCAT, SUBSTRING, UPPER/LOWER, TRIM, REPLACE, LENGTH, INSTR |
| `04-datetime-functions.ipynb` | NOW, CURDATE, DATE_FORMAT, DATEDIFF, DATE_ADD, TIMESTAMPDIFF |
| `05-case-statements.ipynb` | Simple CASE, searched CASE, CASE in aggregations and ORDER BY |
| `06-null-handling.ipynb` | COALESCE, IFNULL, NULLIF, NULL in aggregations, NULL traps |
| `07-aggregations.ipynb` | COUNT/SUM/AVG/MIN/MAX, GROUP BY, HAVING, WHERE vs HAVING |
| `08-joins.ipynb` | INNER JOIN, LEFT JOIN, RIGHT JOIN, multiple JOINs |
| `09-union.ipynb` | UNION, UNION ALL, column matching rules, ORDER BY with UNION |
| `10-subqueries.ipynb` | Subquery in WHERE/SELECT/FROM, EXISTS/NOT EXISTS |
| `11-ctes.ipynb` | WITH clause, multiple CTEs, CTE vs subquery |
| `12-window-functions.ipynb` | OVER, PARTITION BY, ROW_NUMBER/RANK/DENSE_RANK, LAG/LEAD, running totals |
| `13-ddl.ipynb` | CREATE TABLE, ALTER TABLE, DROP, data types, constraints, CHECK |
| `14-normalization.ipynb` | 1NF, 2NF, 3NF, transitive dependencies, denormalization tradeoffs |
| `15-views.ipynb` | CREATE VIEW, DROP VIEW, updatable views, Sakila built-in views |
| `16-stored-procedures.ipynb` | CREATE PROCEDURE, IN/OUT params, CALL, stored functions |
| `17-acid-transactions.ipynb` | ACID properties, START TRANSACTION, COMMIT, ROLLBACK, SAVEPOINT |
| `18-indexing.ipynb` | B-tree, SHOW INDEX, EXPLAIN, CREATE/DROP INDEX, composite indexes |
| `19-query-optimization.ipynb` | EXPLAIN output, index-breaking patterns, query rewrites, EXPLAIN ANALYZE |

## Key Rules When Editing Notebooks

- **Never pre-fill query cells** — practice cells must stay empty (only `%%sql` + a comment question). The user writes all SQL themselves.
- Use `NotebookEdit` tool to edit `.ipynb` files — the `Edit` tool does not work on notebooks.
- All notebooks use `%%sql` (cell magic) for queries and `%sql` (line magic) for the connection setup.
- `DELIMITER` is a MySQL CLI-only command — it cannot be used in notebook cells or any JDBC-based tool. Schema files with triggers/procedures must be loaded via `docker exec`.

## Generating Exams

When asked to generate an exam: scan the relevant notebook(s), extract the concepts covered, and produce a markdown file in the style of a graded exam with questions across concept, practice, and applied categories. Save to `exams/exam-N.md`.
