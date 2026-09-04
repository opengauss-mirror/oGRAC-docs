# ANALYZE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:05:01.834Z pushedAt=2026-08-14T03:03:31.062Z -->

## Function Description

Collects statistics on attributes of tables and indexes in a database.

## Precautions

- This operation can be performed only when the database is in the `OPEN` state.

- The `SYS` user and database administrators (DBAs) have permissions to collect and delete statistics about all users or objects.

- Common users can collect statistics about their own tables. Users with the `ANALYZE ANY` permission can manage (including collect and delete) statistics about all users except the `SYS` user.

## Syntax

Collect table statistics:

```sql
 ANALYZE { TABLE [ schema_name. ]table_name COMPUTE STATISTICS } [ FOR REPORT [ SAMPLE sample_percent ]]
```

Collect index statistics:

```
 ANALYZE { INDEX [ schema_name. ]index_name { COMPUTE STATISTICS | ESTIMATE STATISTICS sample_percent }}
```

## Parameter Description

- `[ schema_name. ]table_name`: Specifies the name of the table on which statistics are to be collected. The table name must be unique under the same user.

- `COMPUTE STATISTICS`: Collects statistics. This is a fixed syntax.

- `ESTIMATE STATISTICS sample_percent`: Collects statistics by sampling. `sample_percent` specifies the sampling rate. The value is an integer in the range of [0, 100].

- `FOR REPORT`: Generates a statistics validation report that compares the statistics gathered at a given sampling rate with the existing system statistics, and reports the deviation rate. The report is saved to the `opt` directory under `LOG_HOME`. `LOG_HOME` is the log directory set during installation.

- `SAMPLE sample_percent`: Sets the sampling rate for the validation report. The value is an integer in the range of [0, 100]. The default value is `100`.

## Examples

- **Collect statistics on the `student` table under the `tester` user.**

```
-- Delete the table tester.student.
 DROP TABLE IF EXISTS tester.student;

-- Create the table tester.student.
 CREATE TABLE tester.student (student_id INT, student_name CHAR(100) NOT NULL, class_name VARCHAR(64), birthday_date DATETIME, other_info VARCHAR(100));

-- Collect statistics on tester.student.
 ANALYZE TABLE tester.student COMPUTE STATISTICS;
 ANALYZE TABLE tester.student COMPUTE STATISTICS FOR REPORT;
 ANALYZE TABLE tester.student COMPUTE STATISTICS FOR REPORT SAMPLE 10;

-- Create an index.
 CREATE INDEX tester.idx on tester.student (student_id);

-- Collect statistics on tester.idx.
 ANALYZE INDEX tester.idx COMPUTE STATISTICS;
 ANALYZE INDEX tester.idx ESTIMATE STATISTICS 10;
```