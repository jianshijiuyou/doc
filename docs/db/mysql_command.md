# 数据库管理语句

| 命令 | 描述 |
|:--------|:------------
| `show engines;` | 查看当前数据库支持的引擎和默认的数据库引擎
| `show variables like "%character%";` | 查看当前字符编码
| `show databases;` | 查看所有数据库
| `use {database name} ;` | 选择数据库
| `show tables;` | 查看当前数据库的所有表
| `show create database {database name};` | 查看数据库的定义
| `show processlist;` | 查看所有客户端的连接情况

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

# 知识点

## 关于连接

客户端如果太长时间没动静，连接器就会自动将它断开。这个时间是由参数 `wait_timeout` 控制的，默认值是 8 小时。

如果在连接被断开之后，客户端再次发送请求的话，就会收到一个错误提醒： Lost connection to MySQL server during query。这时候如果你要继续，就需要重连，然后再执行请求了。