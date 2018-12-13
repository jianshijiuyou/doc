# 数据库管理语句

| 命令 | 描述 |
|:--------|:------------
| `show engines;` | 查看当前数据库支持的引擎和默认的数据库引擎
| `show variables like "%character%";` | 查看当前字符编码
| `show databases;` | 查看所有数据库
| `use {database name} ;` | 选择数据库
| `show tables;` | 查看当前数据库的所有表
| `show create database {database name};` | 查看数据库的定义

# 数据定义语句

## CREATE DATABASE 语法

``` sql
CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name
    [create_specification] ...

create_specification:
    [DEFAULT] CHARACTER SET [=] charset_name
  | [DEFAULT] COLLATE [=] collation_name
```

如果数据库存在且您未指定 `IF NOT EXISTS` ，则会发生错误。

`CHARACTER SET` 指定字符集

`COLLATE` 指定排序规则

举个栗子

``` sql
mysql root@localhost:sys> create database IF NOT EXISTS school
                       ->     CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
Query OK, 1 row affected
Time: 0.172s
```

# 数据处理语句

# 杂谈

命令尾部用 `\G` ，以列的方式显示每行数据：

``` sql
show variables\G
```