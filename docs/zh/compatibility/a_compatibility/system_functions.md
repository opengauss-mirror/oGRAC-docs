# 系统函数

本节介绍 oGRAC 在 A 兼容模式下提供的 Oracle 兼容系统函数。

普通函数调用支持 Bison 和原生解析器。部分函数附带语法，如 `TO_BINARY_FLOAT`、`TO_BINARY_DOUBLE` 的 `DEFAULT ... ON CONVERSION ERROR` 语法仅支持 A 兼容模式下的 Bison 解析器，详见对应函数描述。

## TO_BINARY_FLOAT<a name="to_binary_float"></a>

### 功能描述

将输入按 IEEE-754 binary32（单精度）格式进行转换和舍入，然后以  REAL 类型返回。binary32 有 24 位二进制有效精度，约对应 7 位十进制有效数字。

### 语法格式

```sql
TO_BINARY_FLOAT(expr [, fmt [, nlsparam]])
```

### 参数说明

| 参数 | 说明 |
| --- | --- |
| `expr` | 待转换的数值、字符或 BOOLEAN 表达式。BOOLEAN 的 `TRUE`、`FALSE` 分别转换为 1、0。 |
| `fmt` | 可选的数字输入格式模型。接受字符或数值表达式；数值参数先隐式转换为字符串，再作为格式模型解析。例如数值 `999` 与字符串 `'999'` 等效。指定 `fmt` 时，`expr` 必须为字符类型或字符兼容的数据类型。 |
| `nlsparam` | 可选的字符表达式，目前仅支持设置本次调用的 `NLS_NUMERIC_CHARACTERS`，不修改会话或数据库配置。不能用数值参数代替。 |

`nlsparam` 的写法为 `'NLS_NUMERIC_CHARACTERS=''dg'''`，其中 `d` 是小数点字符，`g` 是分组字符，两者必须是不同的单字节字符。未指定时，小数点为 `.`，分组字符为 `,`。

支持的格式模型子集如下，字母格式元素不区分大小写：

| 元素 | 说明 |
| --- | --- |
| `9`、`0` | 数字占位，用于约束整数部分和小数部分的位数。 |
| `D` | 使用 `NLS_NUMERIC_CHARACTERS` 指定的小数点。 |
| `G` | 使用 `NLS_NUMERIC_CHARACTERS` 指定的分组字符，位于整数部分。 |
| `.`、`,` | 使用固定的小数点 `.`、分组字符 `,`。 |
| `FM` | 支持放在格式模型开头。 |
| `S` | 支持前置符号格式；若同时使用 `FM`，`S` 位于 `FM` 之后。 |
| `EEEE` | 科学计数法，必须位于格式模型末尾。 |

### 返回值和注意事项

- 返回现有 REAL 浮点类型。结果描述可显示为 `BINARY_DOUBLE`。
- 从 binary32 扩展到 REAL 不会恢复舍入时丢失的精度。函数返回后的运算、比较、存储和客户端显示沿用 oGRAC 的 REAL 规则。
- 参数通过语法和类型校验后，`expr` 为 `NULL` 时返回 `NULL`；显式传入的 `fmt` 或 `nlsparam` 为 `NULL` 时也返回 `NULL`。A 兼容模式下的空字符串按 `NULL` 处理。
- 未指定格式模型时，支持普通十进制、科学计数法，以及不区分大小写、可带正负号的 `NaN`、`Inf`、`Infinity`。
- 转换上溢返回同符号的无穷大；下溢按舍入结果返回次正规数或零。转换结果为正零或负零时统一返回正零。
- 非法数字文本、非法格式模型、非法 NLS 参数或不支持的数据类型会报错。

### 示例

```sql
-- 基本转换。
SELECT TO_BINARY_FLOAT('1.25') AS result FROM SYS.SYS_DUMMY;
-- 1.25

-- 数字格式参数先隐式转换为字符串 '999'。
SELECT TO_BINARY_FLOAT('123', 999) AS result FROM SYS.SYS_DUMMY;
-- 123

-- 指定分组符和小数点。
SELECT TO_BINARY_FLOAT('1.234,5', '9G999D9',
    'NLS_NUMERIC_CHARACTERS='',.''') AS result FROM SYS.SYS_DUMMY;
-- 1234.5

SELECT TO_BINARY_FLOAT('1.25E+2', '9D99EEEE') AS result FROM SYS.SYS_DUMMY;
-- 125

SELECT TO_BINARY_FLOAT('3.4028236E38') AS result FROM SYS.SYS_DUMMY;
-- 正无穷大

SELECT TO_BINARY_FLOAT('1E-50') AS result FROM SYS.SYS_DUMMY;
-- 0

-- 函数返回后按现有 REAL 精度做加法，不再次舍入为 binary32。
SELECT TO_BINARY_FLOAT('16777216') + 1 AS result FROM SYS.SYS_DUMMY;
-- 16777217
```

### DEFAULT ... ON CONVERSION ERROR<a name="to_binary_float_default"></a>

该语法为转换失败指定默认值，**仅在 A 兼容模式且 `use_bison_parser=TRUE` 时支持，原生解析器不支持**。

```sql
TO_BINARY_FLOAT(expr DEFAULT default_value ON CONVERSION ERROR
                [, fmt [, nlsparam]])
```

- `default_value` 允许使用字面量、`NULL` 或绑定变量，不支持算术表达式、函数调用等复杂表达式。
- 未指定 `fmt` 时，默认值可为数值、字符或 BOOLEAN；指定 `fmt` 时，默认值必须是兼容字符类型的输入或 `NULL`，并使用与 `expr` 相同的 `fmt`、`nlsparam` 转换。
- 只有 `expr` 的值转换为目标浮点值失败时，改用 `default_value`转换为目标浮点数；上溢、下溢等场景属于成功转换，不触发默认值;`expr` 为 `NULL` 时返回 `NULL`，不使用默认值。
- 该语法不捕获 `expr` 自身的求值错误（例如 `1/0`）、`fmt`/`nlsparam` 的求值或解析错误，以及参数语法、类型校验错误。

本语法需开启Bison语法解析器，由具备权限的用户执行以下 oGRAC 配置语句。

```sql
SELECT NAME, VALUE FROM SYS.DV_PARAMETERS WHERE NAME = 'USE_BISON_PARSER';
ALTER SYSTEM SET use_bison_parser = TRUE SCOPE = MEMORY;
```

在已开启 Bison 的 A 兼容实例中执行：

```sql
SELECT TO_BINARY_FLOAT('bad' DEFAULT 0 ON CONVERSION ERROR) AS result
FROM SYS.SYS_DUMMY;
-- 0

SELECT TO_BINARY_FLOAT('123' DEFAULT NULL ON CONVERSION ERROR, 999) AS result
FROM SYS.SYS_DUMMY;
-- 123

-- '0' 只提供一位整数占位，'123' 与格式不匹配，返回默认值 NULL。
SELECT TO_BINARY_FLOAT('123' DEFAULT NULL ON CONVERSION ERROR, 0) AS result
FROM SYS.SYS_DUMMY;
-- NULL

SELECT TO_BINARY_FLOAT('bad' DEFAULT '1,234.5' ON CONVERSION ERROR,
    '9G999D9', 'NLS_NUMERIC_CHARACTERS=''.,''') AS result FROM SYS.SYS_DUMMY;
-- 1234.5

-- 上溢为成功转换，不返回默认值 0。
SELECT TO_BINARY_FLOAT('3.5E38' DEFAULT 0 ON CONVERSION ERROR) AS result
FROM SYS.SYS_DUMMY;
-- 正无穷大
```

以下调用仍然报错：

```sql
-- 输入表达式求值时除零，尚未进入转换过程。
SELECT TO_BINARY_FLOAT(1 / 0 DEFAULT 0 ON CONVERSION ERROR) FROM SYS.SYS_DUMMY;

-- 数字 fmt 被转换为 '123'，其中包含不支持的格式元素。
SELECT TO_BINARY_FLOAT('123' DEFAULT NULL ON CONVERSION ERROR, 123) FROM SYS.SYS_DUMMY;

-- 默认值本身也不能转换为数字。
SELECT TO_BINARY_FLOAT('bad' DEFAULT 'still_bad' ON CONVERSION ERROR) FROM SYS.SYS_DUMMY;
```

## TO_BINARY_DOUBLE<a name="to_binary_double"></a>

### 功能描述

将输入按 IEEE-754 binary64（双精度）格式进行转换，以  REAL 类型返回。binary64 有 53 位二进制有效精度，约对应 15～17 位十进制有效数字。

### 语法格式

```sql
TO_BINARY_DOUBLE(expr [, fmt [, nlsparam]])
```

### 参数、返回值和注意事项

- `expr`、`fmt`、`nlsparam` 的含义、输入类型限制、数字 `fmt` 的隐式字符串转换，以及格式模型子集与 [TO_BINARY_FLOAT](#to_binary_float) 相同。
- 返回 REAL，存储宽度为 8 字节；结果描述可显示为 `BINARY_DOUBLE`。转换结果后的计算和比较遵从现有 REAL 规则。
- `NULL`、NaN、无穷大、上溢、下溢和零值处理规则与 `TO_BINARY_FLOAT` 相同，但范围和舍入精度使用 binary64。
- 同样支持 `DEFAULT ... ON CONVERSION ERROR`，默认值语法和限制见 [TO_BINARY_FLOAT 的转换失败默认值语法](#to_binary_float_default)；该语法也仅支持 A 兼容模式下的 Bison 解析器。


### 示例

```sql
SELECT TO_BINARY_DOUBLE('1.25') AS result FROM SYS.SYS_DUMMY;
-- 1.25

SELECT TO_BINARY_DOUBLE('123', 999) AS result FROM SYS.SYS_DUMMY;
-- 123

-- 比较数值关系，避免大整数的默认显示位数影响判断。
SELECT CASE WHEN TO_BINARY_DOUBLE('9007199254740993') =
                 TO_BINARY_DOUBLE('9007199254740992')
    THEN 'PASS' ELSE 'FAIL' END AS result FROM SYS.SYS_DUMMY;
-- PASS

SELECT TO_BINARY_DOUBLE('1.8E308') AS result FROM SYS.SYS_DUMMY;
-- 正无穷大

SELECT TO_BINARY_DOUBLE('1E-400') AS result FROM SYS.SYS_DUMMY;
-- 0
```

## COSH<a name="cosh"></a>

### 功能描述

返回参数的双曲余弦，定义为 `COSH(x) = (exp(x) + exp(-x)) / 2`。

### 语法格式

```sql
COSH(n)
```

### 参数、返回值和注意事项

- `n` 为数值、可转换为数值的字符串或 BOOLEAN。
- REAL 输入返回 REAL；NUMBER、整数、字符串、BOOLEAN 输入返回 NUMBER。
- 参数为 `NULL` 时返回 `NULL`。不能转换为数值的字符串报错。
- NUMBER 场景要求参数绝对值不超过 `282.57858863455690819872393985054486897`，超过时报数值溢出。

### 示例

```sql
SELECT COSH(0) AS result FROM SYS.SYS_DUMMY;
-- 1

SELECT COSH(1) AS result FROM SYS.SYS_DUMMY;
-- 约 1.5430806348152437784779056207570616826，返回 NUMBER

SELECT COSH(TO_BINARY_DOUBLE('1')) AS result FROM SYS.SYS_DUMMY;
-- 约 1.54308063481524，返回 REAL

SELECT COSH(TO_BINARY_DOUBLE('711')) AS result FROM SYS.SYS_DUMMY;
-- 正无穷大
```

## SINH<a name="sinh"></a>

### 功能描述

返回参数的双曲正弦，定义为 `SINH(x) = (exp(x) - exp(-x)) / 2`。

### 语法格式

```sql
SINH(n)
```

### 参数、返回值和注意事项

- `n` 为数值、可转换为数值的字符串或 BOOLEAN。
- REAL 输入返回 REAL；NUMBER、整数、字符串、BOOLEAN 输入返回 NUMBER。
- 参数为 `NULL` 时返回 `NULL`。不能转换为数值的字符串报错。
- NUMBER 路径要求参数绝对值不超过 `282.58006617333443970665056803202391465`，超过时报数值溢出。

### 示例

```sql
SELECT SINH(0) AS result FROM SYS.SYS_DUMMY;
-- 0

SELECT SINH(1) AS result FROM SYS.SYS_DUMMY;
-- 约 1.17520119364380145688238185059560081516，返回 NUMBER

SELECT SINH(TO_BINARY_DOUBLE('1')) AS result FROM SYS.SYS_DUMMY;
-- 约 1.1752011936438，返回 REAL

SELECT SINH(TO_BINARY_DOUBLE('-711')) AS result FROM SYS.SYS_DUMMY;
-- 负无穷大
```

## NANVL<a name="nanvl"></a>

### 功能描述

当 `value` 为 NaN（Not a Number，非数值）时返回 `replacement`，否则返回 `value`。

### 语法格式

```sql
NANVL(value, replacement)
```

### 参数、返回值和注意事项

- 两个参数均接受数值、可转换为数值的字符串或 BOOLEAN。
- 任一参数为 REAL 时返回 REAL，否则返回 NUMBER。实际返回的值转换为该共同类型；绑定参数按本次执行的实际类型推导。
- `value` 不是 NaN 或为 `NULL` 时，不执行`replacement`表达式。 `value` 为 NaN 时才执行 `replacement`表达式；被选中的替代值为 `NULL` 时返回 `NULL`，转换失败时报错。

### 示例

```sql
SELECT NANVL(12.5, 0) AS result FROM SYS.SYS_DUMMY;
-- 12.5

SELECT NANVL(TO_BINARY_DOUBLE('NaN'), 0) AS result FROM SYS.SYS_DUMMY;
-- 0，返回 REAL

SELECT NANVL(NULL, 0) AS result FROM SYS.SYS_DUMMY;
-- NULL

-- 首参数不是 NaN，不执行替代值中的转换。
SELECT NANVL(10, TO_NUMBER('bad')) AS result FROM SYS.SYS_DUMMY;
-- 10

SELECT NANVL(TO_BINARY_DOUBLE('NaN'), TO_NUMBER('bad')) FROM SYS.SYS_DUMMY;
-- 数字转换错误
```

## REMAINDER<a name="remainder"></a>

### 功能描述

返回 `dividend` 除以 `divisor` 的余数。计算规则为 `x - y * N`，其中 `x` 为被除数，`y` 为除数，`N` 为最接近 `x / y` 的整数；当商恰好位于两个整数中间时取偶数（ties-to-even）。该规则不同于 `MOD` 的截断余数，正数相除也可能得到负余数。

### 语法格式

```sql
REMAINDER(dividend, divisor)
```

### 参数、返回值和注意事项

- 两个参数均接受数值、可转换为数值的字符串或 BOOLEAN。
- 任一参数为 REAL 时，两个参数统一转换为 REAL 计算并返回 REAL；否则按 NUMBER 计算并返回 NUMBER。
- 任一参数为 `NULL` 时返回 `NULL`；第一个参数为 `NULL` 时不执行第二个参数表达式，但第二个参数仍须通过语法和静态类型校验。
- NUMBER 场景下，除数为零时报除零错误；
- REAL 场景下，除数为零、被除数为无穷大或任一参数为 NaN 时返回 NaN；有限被除数除以无穷大时返回原被除数；零结果统一返回正零。

### 示例

```sql
SELECT REMAINDER(11, 4) AS result FROM SYS.SYS_DUMMY;
-- -1

SELECT REMAINDER(7, 2) AS result FROM SYS.SYS_DUMMY;
-- -1：商为 3.5，取偶数 4，结果为 7 - 2 * 4

SELECT REMAINDER(5, 2) AS result FROM SYS.SYS_DUMMY;
-- 1：商为 2.5，取偶数 2，结果为 5 - 2 * 2

SELECT REMAINDER(5, 0) FROM SYS.SYS_DUMMY;
-- NUMBER 路径，除零错误

SELECT REMAINDER(TO_BINARY_DOUBLE('5'), 0) AS result FROM SYS.SYS_DUMMY;
-- REAL 路径，NaN

SELECT REMAINDER(CAST(NULL AS NUMBER), TO_NUMBER('bad')) AS result FROM SYS.SYS_DUMMY;
-- NULL，不执行第二参数的转换
```

## INITCAP<a name="initcap"></a>

### 功能描述

将每个单词的首字符（若为字母）转换为大写，其余字母转换为小写。字母和数字均属于单词字符；空白、标点、下划线等非字母数字字符作为分隔符，并原样保留。

### 语法格式

```sql
INITCAP(expr)
```

### 参数、返回值和注意事项

- `expr` 为字符表达式，其他支持隐式字符转换的输入会先转换为字符串。
- CHAR 输入返回 CHAR；其他输入返回字符串类型（VARCHAR）。
- 参数为 `NULL` 或 空字符串时返回 `NULL`。
- 数字不改变，但会占据单词的首字符位置。
- 支持多字节字符的简单大小写映射。

### 示例

```sql
SELECT INITCAP('the soap') AS result FROM SYS.SYS_DUMMY;
-- The Soap

SELECT INITCAP('hello-world_sql') AS result FROM SYS.SYS_DUMMY;
-- Hello-World_Sql

SELECT INITCAP('oracle''s') AS result FROM SYS.SYS_DUMMY;
-- Oracle'S

SELECT INITCAP('abc123DEF 99BOTTLES x9-y2') AS result FROM SYS.SYS_DUMMY;
-- Abc123def 99bottles X9-Y2

-- 在 UTF8 数据库执行。
SELECT INITCAP('élÈVE') AS result FROM SYS.SYS_DUMMY;
-- Élève
```

## NCHR<a name="nchr"></a>

### 功能描述

将输入向零截断为整数，取低 16 位作为单个 UTF-16 编码单元，再转换为当前数据库字符集中的字符。

### 语法格式

```sql
NCHR(n)
```

### 参数、返回值和注意事项

- `n` 为数值、可转换为数值的字符串或 BOOLEAN；参数为 `NULL` 时返回 `NULL`。
- 范围检查针对向零截断后的整数，合法范围为 `0`～`4294967295`。转换失败或整数超出范围时报错。
- 返回 `VARCHAR(1 CHAR)`。
- 大于 65535 的合法整数按低 16 位映射。例如 `65536` 映射到 U+0000，而 `65537` 映射到 U+0001。
- 不支持独立 UTF-16 代理项：低 16 位落在 `0xD800`～`0xDFFF` 时返回错误。

### 示例

```sql
SELECT NCHR(65) AS result FROM SYS.SYS_DUMMY;
-- A

SELECT NCHR(65.9) AS result FROM SYS.SYS_DUMMY;
-- A，向零截断为 65

-- 在 UTF8 数据库执行。
SELECT NCHR(20320) AS result FROM SYS.SYS_DUMMY;
-- 你

SELECT RAWTOHEX(NCHR(65536)) AS result FROM SYS.SYS_DUMMY;
-- 00（UTF8 编码），不是 NULL

SELECT NCHR(-1) FROM SYS.SYS_DUMMY;
-- 参数范围错误

SELECT NCHR(55296) FROM SYS.SYS_DUMMY;
-- 55296 即 0xD800，不支持独立 UTF-16 代理项
```
