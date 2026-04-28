# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A personal SQL learning repo. The user is learning MySQL from scratch, progressing through topics in notebook order. Claude acts as tutor — explaining concepts, answering questions about queries, and generating exams.

## Database

**Sakila** — MySQL's official DVD rental sample database. Loaded into a Docker container.

```bash
# Start container (credentials: root / root)
docker start sql-practice

# Load schema (CLI only — DELIMITER syntax does not work via JDBC/notebooks)
docker exec -i sql-practice mysql -u root -proot < sakila-schema.sql

# Interactive shell
docker exec -it sql-practice mysql -u root -proot sakila
```

**Connection string used in all notebooks:**
```
mysql+pymysql://root:root@127.0.0.1:3306/sakila
```

## Notebook Structure

One `.ipynb` file per topic in the root directory. Each notebook follows the same pattern:
- Setup cells: `%pip install jupysql pymysql` then `%load_ext sql` + `%sql <connection>`
- Concept sections: markdown cell explaining the concept with syntax examples
- Practice cells: `%%sql` magic with a question as a comment — **the user writes the SQL, cells are intentionally empty**

| Notebook | Topic |
|---|---|
| `01-select-basics.ipynb` | SELECT, aliases, computed columns, DISTINCT, ORDER BY, LIMIT/OFFSET |
| `02-filtering.ipynb` | WHERE, AND/OR/NOT, IN, BETWEEN, LIKE, IS NULL |
| `03-aggregations.ipynb` | COUNT/SUM/AVG/MIN/MAX, GROUP BY, HAVING, WHERE vs HAVING |
| `04-joins.ipynb` | INNER JOIN, LEFT JOIN, RIGHT JOIN, multiple JOINs |
| `05-subqueries.ipynb` | Subquery in WHERE/SELECT/FROM, EXISTS/NOT EXISTS |
| `06-acid-transactions.ipynb` | ACID properties, START TRANSACTION, COMMIT, ROLLBACK, SAVEPOINT |
| `07-indexing.ipynb` | B-tree, SHOW INDEX, EXPLAIN, CREATE/DROP INDEX, composite indexes |
| `08-window-functions.ipynb` | OVER, PARTITION BY, ROW_NUMBER/RANK/DENSE_RANK, LAG/LEAD, running totals |
| `09-query-optimization.ipynb` | EXPLAIN output, index-breaking patterns, query rewrites, EXPLAIN ANALYZE |

## Key Rules When Editing Notebooks

- **Never pre-fill query cells** — practice cells must stay empty (only `%%sql` + a comment question). The user writes all SQL themselves.
- Use `NotebookEdit` tool to edit `.ipynb` files — the `Edit` tool does not work on notebooks.
- All notebooks use `%%sql` (cell magic) for queries and `%sql` (line magic) for the connection setup.
- `DELIMITER` is a MySQL CLI-only command — it cannot be used in notebook cells or any JDBC-based tool. Schema files with triggers/procedures must be loaded via `docker exec`.

## Generating Exams

When asked to generate an exam: scan the relevant notebook(s), extract the concepts covered, and produce a markdown file in the style of a graded exam with questions across concept, practice, and applied categories. Save to `exams/exam-N.md`.
