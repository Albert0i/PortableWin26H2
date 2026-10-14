# Exporting Oracle Tables to CSV Using SQL*Plus and a Windows Batch File

## A Practical Guide to Automated Oracle Data Extraction with SQL*Plus

### Introduction

Oracle databases are widely used in enterprise environments, and one of the most common administrative tasks is exporting table data for reporting, backup, migration, testing, or data sharing purposes.

Although Oracle provides powerful utilities such as Data Pump (`expdp` and `impdp`), there are many situations where a simple CSV export is sufficient. Examples include:

- Sending reference data to another team
- Creating data snapshots for testing
- Loading data into Excel
- Transferring lookup tables between environments
- Archiving application configuration data

A convenient way to accomplish this is by using **SQL*Plus** together with a **Windows batch (`.bat`) file**.

This article explains how to create a fully automated export process that:

- Connects to Oracle 19c
- Generates a SQL script dynamically
- Exports multiple tables into CSV files
- Produces minimal console output
- Records start and finish times
- Requires only a single command to execute

---

# The Complete Batch File

The following example exports three Oracle tables:

- `PFIHMD1`
- `PFIHMD2`
- `PFIHMD3`

```bat
@echo off

REM === Oracle Connection ===
set HOST=my-oracle-host
set DB=mypdb
set USERID=albertoi
set PASSWORD=mypassword

REM === Create SQL Script ===
(
echo SET MARKUP CSV ON
echo SET FEEDBACK OFF
echo SET HEADING OFF
echo SET PAGESIZE 0
echo SET TERMOUT OFF
echo SET ECHO OFF
echo SET VERIFY OFF

echo SPOOL employees.csv
echo SELECT * FROM employees;
echo SPOOL OFF

echo SPOOL customers.csv
echo SELECT * FROM customers;
echo SPOOL OFF

echo SPOOL products.csv
echo SELECT * FROM products;
echo SPOOL OFF

echo EXIT
) > export_tables.sql

REM === MAIN ===

set STARTDATE=%DATE%
set STARTTIME=%TIME%

echo.
echo Export started at %STARTDATE% %STARTTIME%

REM === Run Export ===

sqlplus -S %USERID%/%PASSWORD%@%HOST%:1521/%DB% @export_tables.sql

set ENDDATE=%DATE%
set ENDTIME=%TIME%

echo.
echo Export completed at %ENDDATE% %ENDTIME%
echo.
```

---

# Understanding the Execution Flow

The batch file performs four major tasks:

1. Set Oracle connection information
2. Dynamically generate a SQL script
3. Execute the SQL script using SQL*Plus
4. Create multiple CSV files

The entire process is automatic and requires only:

```cmd
export_tables.bat
```

---

# Step 1 - Define Oracle Connection Information

```bat
set HOST=oracle-dev-scan
set DB=pdbdev_srv
set USERID=dcdevdta
set PASSWORD=poonsio$da
```

These variables contain the Oracle connection details used by SQL*Plus.

## HOST

```bat
set HOST=oracle-dev-scan
```

Specifies the Oracle server host name.

Examples:

```bat
set HOST=oracle-dev-scan
set HOST=myoracle.company.com
set HOST=192.168.1.100
```

---

## DB

```bat
set DB=pdbdev_srv
```

Specifies the Oracle service name.

Examples:

```bat
set DB=pdbdev_srv
set DB=ORCL
set DB=IHPROD
```

---

## USERID

```bat
set USERID=dcdevdta
```

The Oracle username used to connect to the database.

---

## PASSWORD

```bat
set PASSWORD=poonsio$da
```

The password associated with the Oracle account.

---

# Step 2 - Dynamically Create a SQL Script

The following construct creates a temporary SQL script:

```bat
(
...
) > export_tables.sql
```

After execution, the script `export_tables.sql` contains:

```sql
SET MARKUP CSV ON
SET FEEDBACK OFF
SET HEADING OFF
SET PAGESIZE 0
SET TERMOUT OFF
SET ECHO OFF
SET VERIFY OFF

SPOOL pfihmd1.csv
SELECT * FROM pfihmd1;
SPOOL OFF

SPOOL pfihmd2.csv
SELECT * FROM pfihmd2;
SPOOL OFF

SPOOL pfihmd3.csv
SELECT * FROM pfihmd3;
SPOOL OFF

EXIT
```

This approach eliminates the need to manually maintain a separate SQL file.

---

# Step 3 - Configure SQL*Plus Output

Several SQL*Plus settings are used to produce clean CSV files.

## SET MARKUP CSV ON

```sql
SET MARKUP CSV ON
```

This tells SQL*Plus to generate comma-separated values.

Without this option:

```text
1001 JOHN
1002 MARY
```

With this option:

```csv
1001,JOHN
1002,MARY
```

This is the key setting that makes CSV export possible.

---

## SET FEEDBACK OFF

```sql
SET FEEDBACK OFF
```

Suppresses messages such as:

```text
125 rows selected.
```

This prevents unwanted text from appearing in output.

---

## SET HEADING OFF

```sql
SET HEADING OFF
```

Suppresses column names.

Without it:

```csv
EMPNO,ENAME
1001,JOHN
1002,MARY
```

With it:

```csv
1001,JOHN
1002,MARY
```

If column headers are desired, use:

```sql
SET HEADING ON
```

instead.

---

## SET PAGESIZE 0

```sql
SET PAGESIZE 0
```

Removes page breaks and report formatting.

This is useful when exporting large datasets.

---

## SET TERMOUT OFF

```sql
SET TERMOUT OFF
```

Suppresses query output on the screen.

Without it, every exported row would scroll through the console window.

Instead, data is written directly to the CSV file.

---

## SET ECHO OFF

```sql
SET ECHO OFF
```

Suppresses SQL statement display.

Without it:

```text
SELECT * FROM pfihmd1;
```

would appear during execution.

---

## SET VERIFY OFF

```sql
SET VERIFY OFF
```

Suppresses substitution-variable messages and keeps execution clean.

---

# Step 4 - Export Data Using SPOOL

SQL*Plus uses the `SPOOL` command to write output to a file.

---

## Export PFIHMD1

```sql
SPOOL pfihmd1.csv

SELECT * FROM pfihmd1;

SPOOL OFF
```

Execution flow:

1. Open `pfihmd1.csv`
2. Execute the query
3. Write rows to the file
4. Close the file

Result:

```text
pfihmd1.csv
```

---

## Export PFIHMD2

```sql
SPOOL pfihmd2.csv

SELECT * FROM pfihmd2;

SPOOL OFF
```

Result:

```text
pfihmd2.csv
```

---

## Export PFIHMD3

```sql
SPOOL pfihmd3.csv

SELECT * FROM pfihmd3;

SPOOL OFF
```

Result:

```text
pfihmd3.csv
```

---

# Why Create Separate CSV Files?

Keeping one table per CSV file offers several benefits.

## Easier Data Analysis

```text
pfihmd1.csv
pfihmd2.csv
pfihmd3.csv
```

Each file contains only one dataset.

---

## Easier Data Import

Most tools expect one table per file:

- Microsoft Excel
- Power BI
- Python Pandas
- SQL Server Import Wizard
- Oracle SQL Developer

---

## Easier Troubleshooting

If one export fails, the remaining files are unaffected.

---

# Understanding the SQL*Plus Command

The actual export is performed by:

```bat
sqlplus -S %USERID%/%PASSWORD%@%HOST%:1521/%DB% @export_tables.sql
```

After variable substitution:

```bat
sqlplus -S dcdevdta/poonsio$da@oracle-dev-scan:1521/pdbdev_srv @export_tables.sql
```

---

# Understanding the Connection String

The connection format is:

```text
username/password@host:port/service
```

Example:

```text
dcdevdta/poonsio$da@oracle-dev-scan:1521/pdbdev_srv
```

Where:

| Component | Value |
|-----------|--------|
| Username | dcdevdta |
| Password | poonsio$da |
| Host | oracle-dev-scan |
| Port | 1521 |
| Service | pdbdev_srv |

---

# What Does the -S Option Mean?

```bat
sqlplus -S
```

The `-S` option means:

```text
Silent Mode
```

SQL*Plus suppresses:

- Banner text
- Version information
- Copyright information
- Connection messages

Without `-S`:

```text
SQL*Plus: Release 19.0.0.0.0

Connected to:
Oracle Database 19c ...
```

With `-S`:

```text
(no banner output)
```

This creates a much cleaner experience for automation.

---

# Recording Start and End Time

Before running the export:

```bat
set STARTDATE=%DATE%
set STARTTIME=%TIME%
```

Example:

```text
Wed 10/14/2026
9:40:11.39
```

Displayed as:

```text
Export started at Wed 10/14/2026 9:40:11.39
```

---

After the export:

```bat
set ENDDATE=%DATE%
set ENDTIME=%TIME%
```

Displayed as:

```text
Export completed at Wed 10/14/2026 9:40:31.99
```

This provides a simple execution audit trail.

---

# Sample Execution

Running the batch file:

```cmd
D:\Export>export_tables.bat
```

Produces:

```text
Export started at Wed 10/14/2026 9:40:11.39

Export completed at Wed 10/14/2026 9:40:31.99
```

Generated files:

```text
export_tables.sql
pfihmd1.csv
pfihmd2.csv
pfihmd3.csv
```

---

# Example CSV Output

Suppose `PFIHMD1` contains:

```text
1001 JOHN
1002 MARY
1003 DAVID
```

The generated CSV might contain:

```csv
1001,JOHN
1002,MARY
1003,DAVID
```

If `SET HEADING ON` is used:

```csv
EMPNO,ENAME
1001,JOHN
1002,MARY
1003,DAVID
```

---

# Improving Performance for Large Tables

For tables containing hundreds of thousands or millions of rows, consider adding:

```sql
SET ARRAYSIZE 5000
```

Example:

```sql
SET ARRAYSIZE 5000
SET MARKUP CSV ON
```

This reduces network round-trips between the Oracle client and database server and can noticeably improve export performance.

---

# Security Considerations

Avoid storing production passwords in plain text whenever possible.

Instead of:

```bat
set PASSWORD=poonsio$da
```

you may prompt the user:

```bat
set /p PASSWORD=Enter Password:
```

or use Oracle Wallet in security-sensitive environments.

---

# Possible Enhancements

The solution can be extended easily.

## Export Additional Tables

```sql
SPOOL customer.csv
SELECT * FROM customer;
SPOOL OFF
```

---

## Export Selected Columns

```sql
SELECT
    usrid,
    usrname,
    crtdate
FROM users;
```

---

## Export Filtered Data

```sql
SELECT *
FROM users
WHERE status = 'ACTIVE';
```

---

## Create Daily Snapshot Files

```sql
SPOOL customer_20261014.csv
```

This allows historical exports to be retained.

---

# Conclusion

SQL*Plus remains one of the simplest and most reliable tools for exporting Oracle data. By combining SQL*Plus with a Windows batch file, developers and database administrators can build a lightweight, repeatable, and fully automated export process.

The solution described in this article:

- Connects directly to Oracle 19c
- Creates SQL scripts dynamically
- Exports multiple tables to CSV
- Produces minimal screen output
- Records execution timestamps
- Requires only a single command to run

For small-to-medium export requirements, this method is often faster to implement and easier to maintain than a full Data Pump or ETL solution.

Whether the goal is data migration, reporting, testing, development support, or simple archiving, SQL*Plus remains a valuable and efficient tool in every Oracle professional's toolkit.
