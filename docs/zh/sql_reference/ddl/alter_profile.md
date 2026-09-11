# ALTER PROFILE

## 功能描述

修改已存在配置文件（Profile）的资源限制和密码策略参数。配置文件用于统一管理数据库用户的资源使用与密码安全策略，ALTER PROFILE 通过列出参数名和取值来修改指定参数。

## 注意事项

- 执行 ALTER PROFILE 语句需要 ALTER PROFILE 权限
- 参数项之间以空白分隔，不使用逗号
- 语句中只需列出需要修改的参数，参数名与 CREATE PROFILE 相同
- 参数设置不当可能导致账户被锁定或密码策略过于严格，请根据实际业务需求合理配置

## 语法格式

```sql
ALTER PROFILE profile_name LIMIT
    parameter1 { UNLIMITED | DEFAULT | value }
    [ parameter2 { UNLIMITED | DEFAULT | value } ]
    ...;
```

## 参数说明

### 基本参数

| 参数名 | 说明 |
| ------ | ---- |
| profile_name | 要修改的配置文件的名称 |
| parameter | 要修改的参数名，与 CREATE PROFILE 支持的参数一致 |

### 参数值选项

| 选项 | 说明 |
| ---- | ---- |
| UNLIMITED | 表示无限制 |
| DEFAULT | 使用系统默认值 |
| value | 具体的数值 |

### 可配置参数列表

| 参数名 | 类型 | 默认值 | 单位 | 说明 |
| ------ | ---- | ------ | ---- | ---- |
| FAILED_LOGIN_ATTEMPTS | 整数 | 10 | 次 | 指定用户登录失败的最大尝试次数，超过该次数后账户将被锁定 |
| PASSWORD_LIFE_TIME | 整数 | 15552000 | 秒 | 指定密码的有效期，超过该时间后密码将过期，用户需要修改密码才能登录 |
| PASSWORD_REUSE_TIME | 整数 | UNLIMITED | 秒 | 指定密码重用前必须经过的天数，如果设置为整数，则必须将 PASSWORD_REUSE_MAX 设置为 UNLIMITED |
| PASSWORD_REUSE_MAX | 整数 | UNLIMITED | 次 | 指定密码重用前必须更改的次数，如果设置为整数，则必须将 PASSWORD_REUSE_TIME 设置为 UNLIMITED |
| PASSWORD_LOCK_TIME | 整数 | 86400 | 秒 | 指定账户锁定的时间长度 |
| PASSWORD_GRACE_TIME | 整数 | 604800 | 秒 | 指定密码过期后的宽限期，在此期间登录会收到警告但仍可登录 |
| SESSIONS_PER_USER | 整数 | UNLIMITED | 个 | 指定每个用户允许的最大并发会话数 |
| PASSWORD_MIN_LEN | 整数 | 8 | 字符 | 指定密码的最小长度 |

## 示例

### 同时修改多个参数

```sql
ALTER PROFILE bison_ddl_profile LIMIT
    FAILED_LOGIN_ATTEMPTS 5
    PASSWORD_GRACE_TIME DEFAULT
    SESSIONS_PER_USER UNLIMITED;
```

### 修改密码有效期和最小长度

```sql
ALTER PROFILE app_user_profile LIMIT
    PASSWORD_LIFE_TIME 7776000
    PASSWORD_MIN_LEN 12;
```

### 将参数恢复为系统默认值

```sql
ALTER PROFILE app_user_profile LIMIT
    FAILED_LOGIN_ATTEMPTS DEFAULT;
```

### 将参数设置为无限制

```sql
ALTER PROFILE app_user_profile LIMIT
    SESSIONS_PER_USER UNLIMITED;
```
