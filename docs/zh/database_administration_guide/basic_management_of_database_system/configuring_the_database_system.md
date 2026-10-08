# 配置数据库系统

oGRAC数据库提供了多种系统参数可供用户进行查询或者修改，从而灵活地应对不同的环境和业务场景。

## 查看数据库系统参数

通过系统视图DV_PARAMETERS，用户可以看到当前数据库参数配置情况，以下是对每个参数的解释：

- name：表示参数名字
- VALUE：表示参数视图中的配置值。对立即生效的参数，内存修改可能同时改变 VALUE 和 RUNTIME_VALUE，不能仅凭 VALUE 判断磁盘参数文件是否已经改变。
- RUNTIME_VALUE：运行时实际使用的值
- DEFAULT_VALUE：参数的默认值
- ISDEFAULT：参数是否使用默认配置项。显式设置后，即使数值重新等于 DEFAULT_VALUE，也可能仍显示 FALSE；判断是否恢复原值时应比较实际值，而不是只检查此标志。
- MODIFIABLE：参数是否可修改。TRUE 不代表修改立即生效，还应检查 EFFECTIVE。
- DESCRIPTION：参数描述
- RANGE：参数的取值范围
- DATATYPE：参数的数据类型
- EFFECTIVE：参数生效方式
    - reboot：表示需要重启才能生效
    - immediately：动态生效
    - reconnect：重新连接后生效

## 配置数据库系统参数

通过ALTER SYSTEM语句，用户可以修改数据库系统参数。

配置参数指令语法格式如下：

```sql
ALTER SYSTEM SET parameter_name = parameter_value [SCOPE = {PFILE | MEMORY | BOTH}];

-- 示例：先查询并记录原值，再临时修改一个立即生效的参数
SHOW PARAMETER XA_SUSPEND_TIMEOUT;

ALTER SYSTEM SET XA_SUSPEND_TIMEOUT = 120 SCOPE = MEMORY;

SHOW PARAMETER XA_SUSPEND_TIMEOUT;

-- 本例修改前的值为 60；实际操作时请使用先前记录的原值
ALTER SYSTEM SET XA_SUSPEND_TIMEOUT = 60 SCOPE = MEMORY;

SHOW PARAMETER XA_SUSPEND_TIMEOUT;
```

其中的SCOPE各个参数含义如下：

- SCOPE=PFILE：修改写入参数文件，当前运行实例不应用文件中的新值，重新启动后读取。
- SCOPE=MEMORY：修改当前实例内存中的参数配置，不保存到参数文件。对立即生效的参数，修改可在当前实例中生效；重启后重新读取参数文件。
- SCOPE=BOTH：同时修改内存中的参数配置和参数文件。具体何时影响运行行为，仍取决于参数的 EFFECTIVE；不能将重启生效参数视为立即生效。

SCOPE 决定修改的保存位置，EFFECTIVE 决定参数的生效方式。执行 ALTER SYSTEM 前应先记录原值和生效方式，避免把设置成功等同于功能已经生效。

对立即生效参数进行临时验证时，可显式使用 SCOPE=MEMORY，读取 VALUE、RUNTIME_VALUE 和 DEFAULT_VALUE，并在验证后恢复原值。如需确认参数文件未被修改，可比较修改前后的文件校验值；不要通过展示整个配置文件暴露其中的敏感信息。

必要时，用户需要有ALTER SYSTEM的系统权限。
