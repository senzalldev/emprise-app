# SQL Database Tools

emprise includes built-in SQL tools for querying databases directly from your conversations. Configure database connections in your config, then the LLM can explore schemas, run queries, and analyze data.

## Supported Drivers

| Driver | Config Value | Connection |
|--------|-------------|------------|
| SQLite | `sqlite` | File path via `path` |
| PostgreSQL | `postgres` | DSN via `dsn` |
| MySQL | `mysql` | DSN via `dsn` |
| SQL Server | `sqlserver` or `mssql` | DSN via `dsn` |

## Configuration

Add databases under `tools.databases` in `~/.emprise/config.yaml`:

```yaml
tools:
  databases:
    local:
      driver: sqlite
      path: ~/data/app.db

    prod:
      driver: postgres
      dsn: postgres://user:pass@db.example.com:5432/mydb?sslmode=require

    analytics:
      driver: mysql
      dsn: user:pass@tcp(mysql.example.com:3306)/analytics

    warehouse:
      driver: sqlserver
      dsn: sqlserver://user:pass@sql.example.com:1433?database=warehouse
```

Environment variables work in paths and DSNs:
```yaml
    prod:
      driver: postgres
      dsn: ${DATABASE_URL}
```

## Tools

### sql_query
Run a SQL query and get results as a Markdown table.

```json
{"db": "local", "query": "SELECT * FROM users LIMIT 10"}
```

Returns:
```
| id | name | email | created_at |
| --- | --- | --- | --- |
| 1 | Alice | alice@example.com | 2024-01-15 |
| 2 | Bob | bob@example.com | 2024-02-20 |

(2 rows)
```

Results are capped at 100 rows with a truncation notice.

### sql_schema
List all tables and their row counts.

```json
{"db": "local"}
```

Returns:
```
| Table | Rows |
| --- | --- |
| users | 1523 |
| orders | 45672 |
| products | 234 |
```

Uses database-appropriate queries:
- SQLite: `sqlite_master`
- PostgreSQL: `information_schema.tables` (public schema)
- MySQL: `information_schema.tables` (current database)
- SQL Server: `INFORMATION_SCHEMA.TABLES` (BASE TABLE type)

### sql_describe
Show a table's columns, types, nullability, defaults, and primary keys.

```json
{"db": "local", "table": "users"}
```

Returns:
```
| Column | Type | Nullable | Default | PK |
| --- | --- | --- | --- | --- |
| id | INTEGER | NO | | PK |
| name | TEXT | NO | | |
| email | TEXT | YES | | |
| created_at | TEXT | YES | CURRENT_TIMESTAMP | |
```

### csv_to_sqlite
Import a CSV file into SQLite for analysis.

```json
{"csv_path": "sales.csv", "table": "sales"}
```

This:
1. Reads the CSV file (header row required)
2. Creates a SQLite database at `sales.db` (next to the CSV)
3. Creates a table with all TEXT columns (named from headers)
4. Inserts all data rows
5. Registers the database so `sql_query` can access it immediately

Returns:
```
Imported 1500 rows into sales.db (table: sales)
Columns: region, quarter, revenue, units

Ready for analysis. Examples:
  sql_query(db="sales", query="SELECT * FROM sales LIMIT 5")
  sql_query(db="sales", query="SELECT COUNT(*) FROM sales")
  sql_query(db="sales", query="SELECT region, COUNT(*) as n FROM sales GROUP BY region ORDER BY n DESC")
```

The table name defaults to the CSV filename (sanitized: spaces/hyphens/dots become underscores, lowercased).

## Safety

### Read-Only by Default
`sql_query` only allows read-only statements. Allowed prefixes:
- `SELECT`
- `SHOW`
- `DESCRIBE`
- `EXPLAIN`
- `PRAGMA`
- `WITH` (CTEs)

Any other statement (INSERT, UPDATE, DELETE, DROP, ALTER, etc.) is rejected.

### Blocked Patterns
The following patterns are explicitly blocked regardless of context:
- `EXEC` / `EXECUTE` — stored procedure execution
- `SP_` / `XP_` — system stored procedures
- `BULK` — bulk operations
- `OPENROWSET` / `OPENDATASOURCE` — external data access

### Connection Pooling
Database connections are pooled and reused across queries. Connections are tested with `Ping()` on first use. All connections are closed when emprise exits.

## Azure AD Auth for SQL Server

For SQL Server with Azure AD authentication, use the appropriate DSN format:

```yaml
tools:
  databases:
    azure-sql:
      driver: sqlserver
      dsn: sqlserver://my-server.database.windows.net:1433?database=mydb&fedauth=ActiveDirectoryDefault
```

The `go-mssqldb` driver supports Azure AD authentication methods.

## Examples

### Explore a database
```
"What tables are in the local database?"
→ sql_schema(db="local")

"Describe the users table"
→ sql_describe(db="local", table="users")

"Show me the 10 most recent orders"
→ sql_query(db="local", query="SELECT * FROM orders ORDER BY created_at DESC LIMIT 10")
```

### Analyze CSV data
```
"Analyze this sales data"
→ csv_to_sqlite(csv_path="sales.csv")
→ sql_query(db="sales", query="SELECT region, SUM(revenue) FROM sales GROUP BY region")
```

### Cross-database workflow
```
"Compare user counts between local and prod"
→ sql_query(db="local", query="SELECT COUNT(*) FROM users")
→ sql_query(db="prod", query="SELECT COUNT(*) FROM users")
```
