# CREATE TABLE PARTITION

## 功能描述

创建分区表。
分区表是将一个逻辑上的大表，在物理存储上分割成多个更小、更易管理的部分（称为"分区"或"子表"）的技术。每个分区可以独立存储、备份、维护和查询，但对用户和应用来说，它仍然像一张完整的表。oGRAC 支持范围分区，列表分区，哈希分区，间隔分区。

## 注意事项

分区表和普通表的属性存在一些不兼容。

- 当前支持RANGE、LIST、HASH、INTERVAL四种分区
- 分区键不超过16个字段
- 支持设置为分区键的列类型：UINT32、UINT64、INTEGER、BIGINT、REAL、NUMBER、NUMBER2、NUMBER3、DECIMAL、DATE、TIMESTAMP、INTERVAL_DS、INTERVAL_YM、CHAR、VARCHAR、STRING、BINARY、RAW
- 最多支持16777216个分区，采用HASH分区时最多支持8388608个分区

## 语法格式

**stmt:**

```sql
CREATE TABLE [IF NOT EXISTS] [schema_name.]table_name
    ({column_def_clause}[,...] [external_constraint][,...])
```

共享语句尾部及各子句的完整定义参见 [CREATE TABLE 共享子句](shared/create_table_common_clauses.md)。

分区表继承普通表的列定义、约束及存储子句。

**using_index_clause:**

```
    USING INDEX [ INITRANS int
                | TABLESPACE tablespace_name
                | LOCAL [({PARTITION partition_name [TABLESPACE tablespace_name | INITRANS int | PCTFREE int | ({SUBPARTITION subpartition_name[TABLESPACE tablespace_name]} [,...] )]}[,...])]
              ] [ ...]
```

**table_attr_clause:**

```
    [column_attr_clause]
    [AUTO_INCREMENT [=] value]
    [table_partition_clause]
```

**column_attr_clause:**

```
    [LOB (LOB_item) STORE AS LOB_segname [(LOB_parameters)]]
    [APPENDONLY {ON|OFF}]
```

**table_partition_clause:**

```
    range_partition_clause
    | list_partition_clause
    | hash_partition_clause
    | interval_partition_clause 
```

**range_partition_clause:**

```
    PARTITION BY RANGE (partition_key[,...]) 
    [SUBPARTITION BY {RANGE|LIST|HASH} (subpartition_key [,...])]
    (range_partition_item[,...])

```

**range_partition_item:**

```
    PARTITION partition_name VALUES LESS THAN ({value | MAXVALUE}[,...]) [physical_properties_clause] [(subpartition_item[,...])]
```

**subpartition_item:**

```
    SUBPARTITION subpartition_name [{VALUES LESS THAN ({value | MAXVALUE}[,...]) | VALUES ({value | DEFAULT}[,...])}] [TABLESPACE tablespace_name]
```

**list_partition_clause:**

```
    PARTITION BY LIST (partition_key[,...])
    [SUBPARTITION BY {RANGE|LIST|HASH} (subpartition_key [,...])]
    (list_partition_item[,...])
```

**list_partition_item:**

```
    PARTITION partition_name VALUES ([value][,...] [DEFAULT]) [physical_properties_clause] [(subpartition_item[,...])]
```

**hash_partition_clause:**

```
    PARTITION BY HASH (partition_key[,...])
    [SUBPARTITION BY {RANGE|LIST|HASH} (subpartition_key [,...])]
    ({hash_partition_item1[,...] | hash_partition_item2})
```

**hash_partition_item1:**

```
    PARTITION partition_name  [physical_properties_clause] [(subpartition_item[,...])]
```

**hash_partition_item2:**

```
    PARTITIONS partition_count (STORE IN (tablespace_name[,...]))
```

**interval_partition_clause:**

```
    PARTITION BY RANGE (partition_key) INTERVAL (value)
    [STORE IN (TABLESPACE tablespace_name[,...])]
    [SUBPARTITION BY {RANGE|LIST|HASH} (subpartition_key [,...])]
    (range_partition_item[,...])
```

**physical_properties_clause:**

```
    segment_attr_clause
    | FORMAT row_format_clause
```

## 参数说明

- [普通表共有的参数](create_table.md#参数说明)
- range_partition_clause: RANGE分区
- list_partition_clause: LIST分区
- hash_partition_clause: HASH分区
- interval_partition_clause: INTERVAL分区
- partition_key: 分区键所在列的集合
- VALUES LESS THAN: RANGE分区的分区键最大值
- SUBPARTITION BY {RANGE|LIST|HASH}: 定义二级分区分区方式
- MAXVALUE：分区特殊定义，最大值
- VALUES(value): LIST分区键值
- VALUES(DEFAULT): DEFAULT分区，默认值所在分区
- PARTITIONS partition_count: HASH分区数，指定后自动创建partition_count个分区，数据均匀分布

## 示例

```
-- 范围分区

CREATE TABLE sales_range (
sale_id NUMBER,
sale_date DATE,
amount NUMBER,
region VARCHAR2(50)
)
PARTITION BY RANGE (sale_date) (
PARTITION sales_q1 VALUES LESS THAN (TO_DATE('2024-04-01', 'YYYY-MM-DD')),
PARTITION sales_q2 VALUES LESS THAN (TO_DATE('2024-07-01', 'YYYY-MM-DD')),
PARTITION sales_q3 VALUES LESS THAN (TO_DATE('2024-10-01', 'YYYY-MM-DD')),
PARTITION sales_q4 VALUES LESS THAN (MAXVALUE)
);

-- 列表分区

CREATE TABLE employees_list (
emp_id NUMBER,
emp_name VARCHAR2(100),
department VARCHAR2(50),
salary NUMBER
)
PARTITION BY LIST (department) (
PARTITION dept_sales VALUES ('SALES', 'MARKETING'),
PARTITION dept_tech VALUES ('IT', 'ENGINEERING'),
PARTITION dept_hr VALUES ('HR', 'ADMIN'),
PARTITION dept_other VALUES (DEFAULT)
);

-- 哈希分区方式1：指定分区名称

CREATE TABLE products_hash (
product_id NUMBER,
product_name VARCHAR2(200),
category VARCHAR2(100)
)
PARTITION BY HASH (product_id) (
PARTITION p1,
PARTITION p2,
PARTITION p3,
PARTITION p4
);

-- 哈希分区方式2：指定分区数量

CREATE TABLE orders_hash (
order_id NUMBER,
order_date DATE,
customer_id NUMBER
)
PARTITION BY HASH (order_id)
PARTITIONS 8;

-- 间隔分区
CREATE TABLE sales_interval (
sale_id NUMBER,
sale_date DATE,
amount NUMBER,
region VARCHAR2(50)
)
PARTITION BY RANGE (sale_date)
INTERVAL (NUMTOYMINTERVAL(1, 'MONTH'))
(
PARTITION sales_historical VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD')),
PARTITION sales_jan_2024 VALUES LESS THAN (TO_DATE('2024-02-01', 'YYYY-MM-DD'))
);

-- 二级分区 RANGE + LIST

CREATE TABLE sales_subpart (
    sale_id      NUMBER,
    sale_date    DATE,
    region       VARCHAR2(20),
    product_type VARCHAR2(30),
    amount       NUMBER
)
PARTITION BY RANGE (sale_date)
SUBPARTITION BY LIST (region)
(
    PARTITION sales_2023 VALUES LESS THAN (DATE '2024-01-01')
    (
        SUBPARTITION north VALUES ('NORTH'),
        SUBPARTITION south VALUES ('SOUTH'),
        SUBPARTITION east VALUES ('EAST'),
        SUBPARTITION west VALUES ('WEST'),
        SUBPARTITION other VALUES (DEFAULT)
    ),
    PARTITION sales_2024 VALUES LESS THAN (DATE '2025-01-01'),
    PARTITION sales_future VALUES LESS THAN (MAXVALUE)
);
```
