# HINT

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:16:52.917Z pushedAt=2026-08-18T04:00:29.259Z -->

Hints appear as special comments with a fixed format and position in the SQL text of an SQL statement, thereby influencing the optimizer's selection of execution plans. They can be used to guide the optimizer in generating more efficient query plans.

## Hint Usage

   (1) A hint must follow the SELECT, UPDATE, INSERT, or DELETE keyword. The hint syntax is as follows:

   ```
   /*+ hint(string) hint(string)... */
   ```

   (2) Parameter description:

   - The plus sign (+) directs the database to interpret the comment as a hint list. It must be placed right after the comment delimiter without any spaces.

   - A hint can be any of the types listed in [Hint Overview](#hint-overview). Spaces between the plus sign and the hint are optional. If multiple hints are included within the /*+*/ structure, at least one space must be used to separate them.

   - `string` represents other comment strings that may be interspersed among the hints.

   (3) Special scenarios:
   In certain cases, the database ignores hints and does not return an error.

   - The hint contains spelling or syntax errors. However, the database considers other correctly specified hints within the same comment.

   - The comment containing the hint does not immediately follow a SELECT, UPDATE, INSERT, or DELETE keyword.

   - The combination of hints conflicts with each other. However, the database considers other non‑conflicting hints within the same comment.

## Hint Overview

All hints are listed by functional category.

**Table 1** Variable description

| Category | Syntax | Description |
| ---- | ---- | ---- |
| Optimizer method | RULE | Hints the optimizer to use RBO mode. |
| Path access | FULL | Instructs the optimizer to use a full table scan on the specified table. |
| Path access | INDEX | Hints the optimizer to use a specified index scan on the specified table. |
| Path access | INDEX_ASC | Hints the optimizer to use an ascending index scan on the specified table. |
| Path access | INDEX_DESC | Hints the optimizer to use a descending index scan on the specified table. |
| Path access | NO_INDEX | Hints the optimizer not to use one or more indexes on the specified table. |
| Join order | LEADING | Hints the optimizer to use the specified table order as the prefix in the execution plan. |
| Join order | ORDERED | Hints the optimizer to join tables in the order they appear in the FROM clause. |
| Join method | USE_NL | Hints the optimizer to use a nested-loop join to connect the specified tables, treating the specified tables as the driven tables. |
| Join method | USE_HASH | Hints the optimizer to use a hash join to connect the specified tables, treating the specified tables as the driven tables. |
| Join method | USE_MERGE | Hints the optimizer to use a merge join to connect the specified tables, treating the specified tables as the driven tables. |
| Others | OPT_PARAM | Hints the optimizer to use this parameter in place of the system parameter when executing this SQL statement. |

## RULE

Hints the optimizer to use RBO mode. The syntax is as follows:

```
/*+ RULE */
```

> **Note:** Using the RULE hint forces the optimizer to use the rule-based approach to execute the statement. However, if the SQL statement uses constructs that are not supported by the rule-based optimizer—such as certain join methods or index types—the hint may not be applicable.

## FULL

Hints the optimizer to perform a full table scan on the specified table. The syntax is as follows:

```
/*+ FULL(tablename1 tablename2 [...]) */
```

> **Note:** This hint performs a full table scan only on the specified table, not on all tables in the query.

> **NOTE**
>
> - tablename1: Table name. Separate multiple table names with spaces.
> - If FULL contains only one parameter, the optimizer applies a full table scan only to that table.
> - If FULL contains multiple parameters, the optimizer applies a full table scan to each table represented by the parameters.

Conflicting hint type:

- INDEX: When FULL and INDEX appear together, the hint becomes invalid. It becomes invalid at the syntax level and does not enter the planning stage.

## INDEX

Hints the optimizer to access data from a specified table via index. The syntax is as follows:

```sql
select /*+ index(tablename1 [index_name1] [index_name2] [...])*/* from table_name where ...
```

> **Note:** This hint takes effect only on the specified table.

> **NOTE**
>
> - tablename1: required parameter, the table name
> - index_name1: optional parameter, an index on tablename1. Separate the table name and index name with a space, and separate multiple index names with spaces.
> - Multiple index names are allowed.

> **NOTE**
>
> - Scenario 1: When the INDEX hint specifies only the table name (without naming a specific index), the optimizer is forced to use an index. It will select the most optimal index from all available indexes on that table. A full table scan is not considered, even if it would be cheaper than the chosen index scan.
> - Scenario 2: When the INDEX hint specifies the table name along with one or more specific index names, the optimizer is restricted to using only those named indexes. All other indexes on the table are ignored, even if they would provide better performance. A full table scan is also excluded from consideration.
> - Scenario 3: If the table does not have any of the indexes specified in the hint, the optimizer will not fall back to other existing indexes that are not named in the hint. Instead, it will use a full table scan. For instance, if the table hint_test1 lacks the t1_idx3 index and no other usable index is specified, the optimizer resorts to a full table scan.
> - Scenario 4: If the indexes specified in the hint do not align with the columns used in the filter conditions, the hinted indexes cannot be used. In such cases, if the hint does not name any other index that covers the filter columns, the optimizer will use a full table scan.

Invalid cases:

- If only an index is specified without a table name, the hint is invalid. It becomes invalid at the syntax level and does not enter the planning stage.

- If the specified table name is not referenced in the SQL statement, the hint is invalid. It becomes invalid at the syntax level and does not enter the planning stage.

Conflicting hint type:

- FULL: When INDEX and FULL appear together, the hint becomes invalid. It becomes invalid at the syntax level and does not enter the planning stage.

## INDEX_ASC

Hints the optimizer to use an ascending‑order range scan on the specified index when accessing data from a table.

```sql
select /*+ index_asc(tablename1 [index_name1] [index_name2] [...])*/* from table_name where ...
```

> **NOTE**
>
> - tablename1: required parameter, the table name
> - index_name1: optional parameter, an index on tablename1. Separate the table name and index name with a space, and separate multiple index names with spaces.
> - Multiple index names are allowed.

> **NOTE**
>
> - Scenario 1: If the hint contains only the table name without specifying any index, the optimizer is forced to use an index and perform an ascending‑order scan.
> - Scenario 2: INDEX_ASC does not conflict with ORDER BY.
> - Scenario 3: INDEX_ASC conflicts with ORDER BY column_name DESC. In this case, the hint is invalid, and the ORDER BY clause takes precedence, resulting in a QUERY SORT ORDER BY operator being used.

## INDEX_DESC

Hints the optimizer to perform a descending‑order index scan on the specified table. If the statement uses an index range scan, it explicitly specifies a descending scan range.

```sql
select /*+ index_desc(tablename1 [index_name1] [index_name2] [...])*/* from table_name where ...
```

> **NOTE**
>
> - tablename1: required parameter, the table name
> - index_name1: optional parameter, an index on tablename1. Separate the table name and index name with a space, and separate multiple index names with spaces.
> - Multiple index names are allowed.

> **NOTE**
>
> - Scenario 1: If the hint contains only the table name without specifying any index, the optimizer is forced to use an index and perform a descending‑order scan.
> - Scenario 2: INDEX_DESC does not conflict with ORDER BY column_name DESC.
> - Scenario 3: INDEX_DESC conflicts with ORDER BY (ASC). In this case, the hint is invalid, and the ORDER BY clause takes precedence, resulting in a QUERY SORT ORDER BY operator being used.

## NO_INDEX

Instructs the optimizer not to use indexes on the specified table. This hint can be used to disable indexes in many queries before dropping unnecessary indexes. The syntax is as follows:

```sql
select /*+ no_index(tablename1 [index_name1] [index_name2] [...])*/* from table_name where ...
```

> **NOTE**
>
> - tablename1: required parameter, the table name
> - index_name1: optional parameter, an index on tablename1. Separate the table name and index name with a space, and separate multiple index names with spaces.
> - Multiple index names are allowed.

> **NOTE**
>
> - Scenario 1: If the hint specifies only the table name and no index name, all indexes on that table are disabled, and the optimizer performs a full table scan.
> - Scenario 2: If specific index names are specified, the hinted indexes are disabled. Other indexes not listed in the hint, such as t1_idx0, are still considered, as well as a full table scan. The optimizer will then choose the best execution plan among the available approaches.

Conflicting hint type:
INDEX: When NO_INDEX and INDEX appear together, the hint becomes invalid. It becomes invalid at the syntax level and does not enter the planning stage.

## LEADING

Tells the optimizer which table's data to access first in a multi-table join query. The syntax is as follows:

```
/*+ LEADING (table_name1 [table_name2 ...]) */
```

> **NOTE**
>
> - tablename1: required parameter, the table name. table_name2: table name. At least one table must be specified as a parameter.

> **NOTE**
>
> - Scenario 1: If multiple tables are specified, the tables are scanned in the order in which they are listed.
> - Scenario 2: Tables not listed in the LEADING hint are scanned after all specified tables. The scan order among the unspecified tables is determined by the optimizer's original plan.
> - Scenario 3: This hint can be used together with join method hints to obtain a better execution plan.

Conflicting hint type:
ORDERED: When LEADING and ORDERED appear together, the hint becomes invalid. It becomes invalid at the syntax level and does not enter the planning stage.

## ORDERED

Instructs the optimizer to use the table order specified in the FROM clause as the join order. The syntax is as follows:

```
/*+ ORDERED */
```

> **NOTE**
>
> - No parameters are required.

> **NOTE**
>
> - Scenario 1: No parameters need to be passed. All tables described in FROM are joined in order, meaning that the scan order of tables in the execution plan matches the order in which the tables appear in FROM.

Conflicting hint type:
LEADING: When LEADING and ORDERED appear together, the hint becomes invalid. It becomes invalid at the syntax level and does not enter the planning stage.

## USE_NL

Instructs the optimizer to use nested-loop joins when joining the specified tables. It only specifies the join method and does not affect the join order of the tables. The syntax is as follows:

```
/*+ USE_NL(table_name1 [table_name2 ...])*/
```

> **NOTE**
>
> - table_name1: table name. It hints the optimizer to use the specified table as the driven table.

> **NOTE**
>
> - Scenario 1: The hint instructs the optimizer to treat the specified tables as driven tables and to join them using nested-loop joins.
> - Scenario 2: If only one table is specified, the optimizer builds the execution plan with that table as the driven table, but the join method is not necessarily a nested-loop join. If two tables are specified, the join between them will be a nested-loop join.

## USE_HASH

Instructs the optimizer to use hash joins when joining the specified tables. During a hash join, if one side of the join is small enough, the hash join can be performed in memory. Therefore, this hint can be applied based on the actual situation. The syntax is as follows:

```
/*+ USE_HASH(table_name1 [table_name2 ...])*/
```

> **NOTE**
>
> - table_name1: table name. It hints the optimizer to use the specified table as the driven table.

> **NOTE**
>
> - Scenario 1: The hint instructs the optimizer to treat the specified tables as driven tables and to join them using hash joins.
> - Scenario 2: If only one table is specified, the optimizer builds the execution plan with that table as the driven table, but the join method is not necessarily a hash join. If two tables are specified, the join between them will be a hash join.

## USE_MERGE

Instructs the optimizer to execute table joins using the merge join method. The syntax is as follows:

```
/*+ USE_MERGE(table_name1 [table_name2 ...])*/
```

> **NOTE**
>
> - table_name1: table name. It hints the optimizer to use the specified table as the driven table.

> **NOTE**
>
> - Scenario 1: The hint instructs the optimizer to treat the specified tables as driven tables and to join them using merge joins.
> - Scenario 2: If only one table is specified, the optimizer builds the execution plan with that table as the driven table, but the join method is not necessarily a merge join. If two tables are specified, the join between them will be a merge join.
