# CREATE TABLE 共享子句

## 共享语句尾部

以下子句在 `CREATE TABLE` 与 `CREATE TABLE PARTITION` 的语法中功能与写法相同：

```sql
[ON COMMIT {DELETE|PRESERVE} ROWS]
[physical_properties_clause]
[table_attr_clause]
[CRMODE PAGE]
```

## 子句展开

**column_def_clause:**

```
    column_name {datatype | SERIAL}
    [DEFAULT expr [ON UPDATE expr]]
    [AUTO_INCREMENT]
    [COMMENT 'comment_str']
    [COLLATE collation_name]
    [col_level_constraint][...]
```

**col_level_constraint:**

```
    CONSTRAINT constraint_name {[NOT] NULL
                                | UNIQUE
                                | PRIMARY KEY
                                | CHECK(expr)
                                | refenence_clause }[...]
```

**refenence_clause:**

```
    REFERENCES [schema_name.]table_name[(column_name)] [ON DELETE {CASCADE | SET NULL}]
```

**external_constraint:**

```
    CONSTRAINT constraint_name {UNIQUE(column_name[,...]) [using_index_clause]
                                | PRIMARY KEY(column_name[,...]) [using_index_clause]
                                | CHECK(expr)
                                | FOREIGN KEY(column_name[,...]) refenence_extend_clause}
```

**using_index_clause:**

```
    USING INDEX [ INITRANS int
                | PCTFREE int
                | TABLESPACE tablespace_name
                | LOCAL
              ]
```

**refenence_extend_clause:**

```
    REFERENCES [schema_name.]table_name[(column_name[,...])] [ON DELETE {CASCADE | SET NULL}]
```

**physical_properties_clause:**

```
    segment_attr_clause
    | ORGANIZATION EXTERNAL external_table_clause
    | FORMAT ({ASF|CSF})
```

**segment_attr_clause:**

```
    { physical_attr_clause | TABLESPACE tablespace_name}[ ...]
```

**physical_attr_clause:**

```
    {
        PCTFREE int
        | INITRANS int
        | MAXTRANS int
        | storage_clause
    }[ ...]
```

**storage_clause:**

```
    STORAGE ({INITIAL int [K|M|G|T] | MAXSIZE {UNLIMITED | int [K|M|G|T]}}[ ...])
```

**external_table_clause:**

```
    ([TYPE LOADER] external_data_properties)
```

**external_data_properties:**

```
    DIRECTORY directory_name
    [ACCESS PARAMETERS (opaque_format_spec)]
    LOCATION location_name
```

**opaque_format_spec:**

```
    RECORDS DELIMITED BY records_delimiter FIELDS TERMINATED BY fields_term
```

**table_attr_clause:**

```
    [column_attr_clause]
    [AUTO_INCREMENT [=] value]
    [AS subquery]
```

**column_attr_clause:**

```
    [LOB (LOB_item) STORE AS {[(LOB_parameters)]}]
    [APPENDONLY {ON|OFF}]
```

**LOB_parameters:**

```
    [TABLESPACE tablespace_name | {ENABLE | DISABLE} STORAGE IN ROW][ ...]
```
