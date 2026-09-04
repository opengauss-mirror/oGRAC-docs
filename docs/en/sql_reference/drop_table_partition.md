# DROP TABLE PARTITION

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-30T12:16:12.284Z pushedAt=2026-08-17T08:38:29.090Z -->

## Function Description

Deletes a specified partition and its data from a partitioned table.

## Precautions

- Executing this statement requires the ALTER TABLE permission on the target table.

- Dropping a partition will also delete all data stored in that partition, and the partition itself cannot be restored from the recycle bin.

- A partitioned table must retain at least one partition. The only remaining partition cannot be dropped.

- If other objects (such as indexes or views) depend on this partition, the related dependencies must be resolved before dropping.

- Dropping partitions is not supported during oGRAC restart recovery.

## Syntax

**stmt:**

```sql
ALTER TABLE [schema_name.]table_name
    DROP PARTITION partition_name
```

## Parameter Description

- **schema_name**: Name of the schema to which the partitioned table belongs. If omitted, the current user is used by default.

- **table_name**: Name of the partitioned table from which the partition is to be dropped.

- **partition_name**: Name of the partition to be dropped.

## Examples

```
-- Drop a partition from a RANGE partitioned table
ALTER TABLE sales_range DROP PARTITION sales_q1;

-- Drop a partition from a LIST partitioned table
ALTER TABLE employees_list DROP PARTITION dept_hr;

-- Drop a partition from a partitioned table in a specific schema
ALTER TABLE hr.sales_range DROP PARTITION sales_2023_q4;
```
