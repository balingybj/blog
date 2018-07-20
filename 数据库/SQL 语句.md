# SQL 语句

本文记录目前可能需要用到的 SQL 语句。大部分摘抄自《MySQL 必知必会》。

## 使用 MySQL 数据库

### 连接到数据库服务器

可以用 mysql 命令行工具连接到 MySQL 数据库服务器。需要知道用户名、服务器 IP、端口号，可能还需要验证身份需要的密码。

也可以用有界面的工具，连接时需要填入的信息也差不多。

### 列出当前服务器中包含的数据库

```mysql
SHOW DATABASE
```

列出的库中有一些是 MySQL 内部使用的数据库，用来保存数据库、表、列、用户、权限等元信息。

### 选择要操作的数据库

```mysql
USE database_name
```

### 列出当前数据库中的表

```mysql
SHOW TABLES
```

### 列出表的所有字段

```mysql
SHOW COLUMNS FROM [tablename]
```

MySQL 支持用 `DESCRIBE` 作为 `SHOW COLUMS FROM` 的快捷命令。

### 查看创建数据库和表时使用的 SQL 语句

```mysql
SHOW CREATE DATABASE database_name
```

```mysql
SHOW CREATE TABLE table_name
```

## 数据查询

数据查询使用 `SELECT` 语句。`SELECT` 语句用于在指定的表格中查询记录，并可以指定想要查询的字段。格式如下:

```mysql
SELECT 字段1,字段2,... FROM 表名;
```

SELECT 后跟想要查询的字段名，如果要查询多个字段，则字段间用逗号隔开。表名表示要查询的表。

如果想要查询所有字段，则字段名用 `*` 代替，一般不建议这么做，最好指定所有想要查询的字段，这样即使数据库新增了字段，查询结果也不会出现应用不能解析的字段。

```mysql
SELECT * FROM 表名;
```

