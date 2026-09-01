# Array Types

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:05:37.887Z pushedAt=2026-08-25T03:42:37.631Z -->

The types of array elements can be the basic data types introduced in this chapter.

## Array Constant Expression

The declaration syntax is as follows:

```
ARRAY [ param ]
```

or

```
'{ param }'
```

Where:

- `param` indicates the element values in the array, which can contain zero or more values. Multiple values must be separated by commas (,). An element with no value can be written as `NULL`.

- The data type of the first element is used as the array type. Therefore, the types of all elements in an array must be the same or be convertible to each other.

> **NOTE:**
>
>- Array types cannot be BINARY, VARBINARY, CLOB, BLOB, CURSOR, RAW, or IMAGE.
>- Multidimensional arrays are not supported.
>- Index, primary key, foreign key, and unique key constraints cannot be created on array-type columns.

## Column Type

When creating a table, you can set a column type to the array type. The syntax is as follows:

```
data_type [(n)]
```

Where:

- `data_type`: indicates the basic data type.

- `n`: indicates the array length.

> **NOTE:**
>
> - The specified array length `n` has no practical effect, as the array will grow automatically. Accessing an out-of-bounds index will return `NULL` rather than raising an error. The maximum length of an array is 2<sup>31</sup> – 1. The array element values are actually stored in a large object (LOB) segment, which supports a maximum size of (4 GB – 1) bytes. Therefore, the number of elements that can be stored depends on the data type of each element.
> - Array elements can be accessed using square brackets ([]), with indexes beginning at 1.

Example:

```
SQL> CREATE TABLE array_t1 (a int[2]);

SQL> insert into array_t1 values (array[1,2]);

1 rows affected.

SQL> select a[3] from array_t1;

A
----------------------------------------


1 rows fetched.

SQL> select a[2] from array_t1;

A
----------------------------------------
2

1 rows fetched.

-- Batch update array elements.
SQL> update array_t1 set a[2:4] = '{3,4,5}';

SQL> select a from array_t1;

A
----------------------------------------------------------------
{1,3,4,5}

1 rows fetched.

SQL> drop table array_t1;
```