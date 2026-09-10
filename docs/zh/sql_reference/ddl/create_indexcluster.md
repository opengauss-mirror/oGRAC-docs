# CREATE INDEXCLUSTER

## 功能描述

在一条语句中为表批量创建多个索引。语句中的每个索引项与 CREATE INDEX 语法一致，可分别指定唯一性、索引列及排序方式、表空间、并行度等属性，适用于一次性创建大量索引的场景。

## 注意事项

- 创建索引需要 CREATE ANY INDEX 权限
- 一条 CREATE INDEXCLUSTER 语句最多创建 8 个索引，超出时报错
- 每个索引项的选项与 CREATE INDEX 一致，支持 TABLESPACE、PARALLEL、NOLOGGING、REVERSE、PCTFREE、INITRANS、CRMODE、ONLINE、LOCAL
- 支持 EXPLAIN [PLAN FOR] CREATE INDEXCLUSTER ... 形式，用于查看执行计划

## 语法格式

```sql
CREATE INDEXCLUSTER (
    [UNIQUE] INDEX [schema_name.]index_name ON [schema_name.]table_name
        ( column_name [ASC|DESC] [, ...] ) [index_options]
    [, ...]
);
```

**index_options:**

```sql
[TABLESPACE tablespace_name]
[PARALLEL n]
[NOLOGGING]
[REVERSE]
[PCTFREE int]
[INITRANS int]
[CRMODE PAGE]
[ONLINE]
[LOCAL]
```

## 参数说明

- **[UNIQUE]**: 可选，指定创建唯一索引
- **index_name**: 索引名称
- **schema_name**: 索引或表所属的 schema，省略时为本用户 schema
- **table_name**: 索引所在的表
- **column_name**: 索引列
- **ASC | DESC**: 索引列的排序方式，ASC 为升序，DESC 为降序
- **TABLESPACE**: 指定索引所在的表空间
- **PARALLEL**: 并行创建索引的并行度
- **NOLOGGING**: 创建索引时不记录 REDO
- **REVERSE**: 反向索引
- **PCTFREE**: 指定索引块中为未来索引条目更新预留的空间百分比，单位%
- **INITRANS**: 索引块的初始事务槽数量
- **CRMODE**: MVCC 模式，PAGE 是页级 MVCC，默认和表的 CRMODE 一致
- **ONLINE**: 在线创建索引
- **LOCAL**: 分区索引，即每个分区上单独创建索引

## 示例

### 一次创建 8 个索引

```sql
create indexcluster (
    INDEX idx_clus_04 on TB_PARTITION_HASH_01(EMPNO ASC,MGR DESC) parallel 48,
    INDEX idx_clus_05 on TB_PARTITION_HASH_01(EMPNO ASC,SAL DESC) parallel 48,
    INDEX idx_clus_06 on TB_PARTITION_HASH_01(JOB ASC,ENAME DESC) parallel 48,
    INDEX idx_clus_07 on TB_PARTITION_HASH_01(ENAME ASC,MGR DESC) parallel 48,
    INDEX idx_clus_08 on TB_PARTITION_HASH_01(MGR ASC,HIREDATE DESC) parallel 48,
    INDEX idx_clus_09 on TB_PARTITION_HASH_01(HIREDATE ASC,SAL DESC) parallel 48,
    INDEX idx_clus_10 on TB_PARTITION_HASH_01(SAL ASC,COMM DESC) parallel 48,
    INDEX idx_clus_11 on TB_PARTITION_HASH_01(HIREDATE ASC,SAL,DEPTNO DESC) parallel 48);
```

### 查看批量创建索引的执行计划

```sql
explain plan for create indexcluster (
    index bison_explain_idx1 on bison_explain_src(id),
    index bison_explain_idx2 on bison_explain_src(val)
);
```
