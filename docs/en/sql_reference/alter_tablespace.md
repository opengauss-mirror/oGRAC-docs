# ALTER TABLESPACE

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-06T07:05:01.699Z pushedAt=2026-08-27T10:35:58.339Z -->

## Function Description

Modifies the attributes of an existing tablespace.

## Precautions

- Only the tablespace owner or users granted the `ALTER TABLESPACE` permission can execute this command. System administrators have this permission by default.

- The following operations can only be performed when the database is in the `OPEN` state:

    - Adding a data file

    - Deleting a data file

    - Modifying the `AUTOEXTEND` attribute of a file

    - Renaming a tablespace

- The following operation can only be performed when the database is in the `OPEN RESTRICTED` state:

    - Modifying the name of a data file

- The `AUTOOFFLINE` attribute can only be set for user tablespaces.

## Syntax

Rename a tablespace.

```sql
 ALTER TABLESPACE 'tablespace_name'
    RENAME TO 'new_tablespace_name'
```

Shrink or punch the tablespace size.

```
 ALTER TABLESPACE 'tablespace_name'
    { SHRINK SPACE KEEP integer [ K | M | G | T ]
    | PUNCH { SIZE integer [ K | M | G ] } }
```

Modify the `AUTOOFFLINE` or `AUTOEXTEND` attribute of a tablespace.

```
 ALTER TABLESPACE 'tablespace_name'
    { AUTOOFFLINE { ON | OFF }
     | AUTOEXTEND 
          { OFF 
          | ON [ NEXT integer [ K | M | G ] ] [ MAXSIZE { integer [ K | M | G ] | UNLIMITED }] 
          }
    }
```

Add a data file to a tablespace or delete a data file from a tablespace.

```
 ALTER TABLESPACE 'tablespace_name'
   { ADD DATAFILE 
      { 'file_name' SIZE integer [ K | M | G ] [COMPRESS] 
        [AUTOEXTEND 
          { OFF 
          | ON [ NEXT integer [ K | M | G ] ] [ MAXSIZE { integer [ K | M | G ] | UNLIMITED }] 
          } 
        ] [ segments integer ]
      }
     | DROP DATAFILE 'file_name' }
```

Rename a data file in a tablespace.

```
 ALTER TABLESPACE 'tablespace_name'
   RENAME DATAFILE 'old_file_name' TO 'new_file_name'
```

## Parameter Description

- **Common parameters**:  
    `integer`: Indicates a positive integer excluding `0`.  
    `K`: Indicates the unit KB.  
    `M`: Indicates the unit MB.  
    `G`: Indicates the unit GB.  
    `T`: Indicates the unit TB.

- `tablespace_name`: Indicates the tablespace to be modified.

    Value range: an existing tablespace name.

- `new_tablespace_name`: Indicates the new name of the tablespace.

    Value range: a string that conforms to the identifier naming convention.

- `RENAME TO 'new_tablespace_name'`: Renames a tablespace.

- `SHRINK SPACE KEEP integer [ K | M | G | T ]`: Shrinks the tablespace size.

    - In the `RESTRICTED` mode with no residual transactions, this command can be used to rebuild the UNDO tablespace.

    - In the `OPEN` mode, any tablespace except TEMP can be shrunk.

    - The TEMP tablespace can be shrunk only in the `OPEN RESTRICTED` mode.

    - The size of a tablespace that can be shrunk depends on the size of each data file's contiguous free space starting from the high water mark in the tablespace.

    - If the user-defined `KEEP` _size_ is smaller than the actual reservable space, the user-defined size takes precedence. Otherwise, the actual reservable space takes precedence.

    - The `SHRINK` operation, if interrupted, may lead to residual data.

    Value range: `1M` to `8000T`

- `PUNCH { SIZE integer [ K | M | G ] }`: Punches idle pages in the tablespace.

    - The encrypted, temporary, UNDO, and default tablespaces cannot be punched.

    - Punched pages cannot be reused by the original tablespace.

    Value range: `1M` to `500G`

- `AUTOOFFLINE { ON | OFF }`: Enables or disables the automatic offline function for the tablespace.

    - For a tablespace with `AUTOOFFLINE` set to `ON`, during database startup, any file open failure will cause the tablespace to be automatically taken offline. However, after startup, such exceptions will not trigger automatic offline.

    - For a tablespace with `AUTOOFFLINE` set to `ON`, if a file is damaged or other faults occur during database startup, you can load the database to the `MOUNT` state. If `AUTOOFFLINE` is set to `OFF`, the database cannot be started when a file in the tablespace is damaged or other faults occur.

- `AUTOEXTEND { OFF | ON [ NEXT integer [ K | M | G ] ] [ MAXSIZE { integer [ K | M | G ] | UNLIMITED }] }`: Sets the automatic extension attribute for the tablespace or data file.

    - If `AUTOEXTEND` is not specified or `AUTOEXTEND` is set to `OFF`, automatic extension is disabled by default.

    - **If `AUTOEXTEND` is set to `ON`, the following attributes can be configured**:

      - `NEXT` indicates the extension size. If it is not specified, the default value 16 MB is used.

      - `MAXSIZE` indicates the extension upper limit. If it is not set or is set to `UNLIMITED`, the extension upper limit for the UNDO tablespace is 32 GB, and that for other tablespaces is 8 TB. The user-specified upper limit cannot exceed this range.

      - If both `MAXSIZE` and `NEXT` are set, the value of `MAXSIZE` must be no less than that of `NEXT`.

- `ADD DATAFILE { 'file_name' SIZE integer [ K | M | G ] [COMPRESS] [AUTOEXTEND ...] [ segments integer ] }`: Adds a data file to the tablespace.

    - `file_name` indicates the data file name. If a relative path is specified, the file is stored in the `data` directory under the data directory by default.

    - `COMPRESS` specifies the new data file as a compressed file. A compressed file is used to store tables with compression attributes. When the table compression feature is used, a compressed file needs to be created in the corresponding tablespace.

    - For details about `AUTOEXTEND`, see the preceding description.

    - `segments integer` indicates the number of segments to be extended. The `SEGMENTS` clause can only be used for the UNDO tablespace in the `RESTRICTED` mode. When it is used, only one data file can be added at a time. The lower limit of the `SEGMENTS` clause is `1`. Before the value of `_UNDO_SEGMENTS` reaches the upper limit `1024`, the upper limit of the clause is the value of `1024` minus the value of `_UNDO_SEGMENTS`. If the value of `_UNDO_SEGMENTS` is already `1024`, extension cannot be performed. The `SEGMENTS` clause is a remedial measure used in the `RESTRICTED` mode. It involves multiple resource persistence steps. Therefore, the atomicity of operations cannot be guaranteed during the entire process. If an exception occurs when the `SEGMENTS` clause is used, you need to handle it as required.

    - The mechanism of UNDO SEGMENTS extension is incompatible with that of UNDO SPACE switching. Therefore, if UNDO SEGMENTS extension is performed before UNDO SPACE switching, an error is reported.

- `RENAME DATAFILE 'old_file_name' TO 'new_file_name'`: Renames a data file in the tablespace.

## Examples

- **Add data files to the tablespace `tbs_student`.**

```
-- Create the tablespace tbs_student.
 CREATE TABLESPACE tbs_student DATAFILE '-dfile_tbs_01' SIZE 32M AUTOEXTEND ON NEXT 10M;

-- Add the data files my_datafile (size: 32M), manager_dfile (size: 32M), and section_dfile (size: 32M) to the tablespace tbs_student.
 ALTER TABLESPACE tbs_student ADD DATAFILE '-my_datafile' SIZE 32M, 'manager_dfile' SIZE 32M, 'section_dfile' SIZE 32M;

```

- **Delete the data file `manager_dfile` from the tablespace `tbs_student`.**

```
ALTER TABLESPACE tbs_student DROP DATAFILE 'manager_dfile';
```

- **Rename the data file `my_datafile` in the tablespace `tbs_student` to `new_my_datafile` in the `OPEN RESTRICTED` state.**

```
-- Rename the data file my_datafile in the tablespace tbs_student to new_my_datafile in the OPEN RESTRICTED state.
ALTER TABLESPACE tbs_student RENAME DATAFILE 'my_datafile' TO 'new_my_datafile';
```

- **Enable automatic extension for the tablespace `tbs_student` so that the tablespace can be automatically expanded when it is full. You can manually specify the extension size.**

```
ALTER TABLESPACE tbs_student AUTOEXTEND ON NEXT 5M;
```

- **Rename the tablespace `tbs_student` to `data_tbs_student`.**

```
ALTER TABLESPACE tbs_student RENAME TO data_tbs_student;
```