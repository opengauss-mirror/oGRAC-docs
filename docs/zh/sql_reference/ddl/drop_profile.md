# DROP PROFILE

## 功能描述

`DROP PROFILE` 语句用于删除数据库中已存在的配置文件（Profile）。配置文件是用于管理数据库用户资源限制和密码策略的集合。

## 注意事项

执行 DROP PROFILE 语句的用户需具备 DROP PROFILE 系统权限。DEFAULT PROFILE 为系统默认配置文件，不允许删除。若配置文件正被其他用户使用，需使用 CASCADE 选项将其关联用户重置为默认配置文件后再删除。

## 语法格式

```sql
DROP PROFILE profile_name [CASCADE];
```

## 参数说明

- **profile_name**: 要删除的配置文件名称。

- **CASCADE**: 可选参数。如果配置文件正在被用户使用，使用此选项会将这些用户的配置文件重置为默认配置文件，然后删除该配置文件。如果不指定此选项且配置文件正在被使用，则删除操作会失败。

## 示例

### 示例 1：删除未被使用的配置文件

```
DROP PROFILE APP_USER_PROFILE;
```

### 示例 2：使用 CASCADE 删除正在被使用的配置文件

```
DROP PROFILE APP_USER_PROFILE CASCADE;
```
