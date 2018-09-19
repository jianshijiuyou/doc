## CentOS 安装 MySQL

### 安装

1.检测系统是否已经安装过 mysql 或其依赖，若已装过要先将其删除，否则第 4 步使用 yum 安装时会报错：

```
>>> yum list installed | grep mysql
mysql-libs.i686         5.1.71-1.el6      @anaconda-CentOS-201311271240.i386/6.5
>>> yum -y remove mysql-libs.i686
```

2.从 mysql 的[官网](https://dev.mysql.com/downloads/repo/yum/)下载对应的 `rpm` 文件（这里我是 centos 7，所以下载 `mysql80-community-release-el7-1.noarch.rpm`）：

```
wget https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
```


3.安装第一步下载的 rpm 文件：

```
yum install mysql80-community-release-el7-1.noarch.rpm 
```

安装成功后，我们可以看到 `/etc/yum.repos.d/` 目录下增加了以下两个文件

```
>>> ls /etc/yum.repos.d/ | grep mysql
mysql-community-source.repo
mysql-community.repo
```

查看 mysql57（或你想安装的其他版本） 的安装源是否可用，如不可用请自行修改配置文件（`/etc/yum.repos.d/mysql-community.repo`）使 mysql57 下面的 `enable=1`

若有 mysql 其它版本的安装源可用，也请自行修改配置文件使其 `enable=0`

!> 总之把你想安装的版本下面修改成 `enable=1` 就行了

```
[root@localhost Downloads]# yum repolist enabled | grep mysql
mysql-connectors-community/x86_64       MySQL Connectors Community           51
mysql-tools-community/x86_64            MySQL Tools Community                63
mysql80-community/x86_64                MySQL 8.0 Community Server           17
```

可以看到，这里默认只能安装 `MySQL 8.0`，如果你想安装 `MySQL 5.7`，请把 `mysql-community.repo` 文件中 `[mysql57-community]` 下面修改成 `enable=1`，把 `[mysql80-community]` 下面修改成 `enable=0` 。

4.使用 yum 安装 mysql：

```
yum install mysql-community-server
```


5.启动 mysql 服务：

```
service mysqld start

# or

systemctl start mysqld
```

### 修改密码

查看 root 密码：


```
[root@localhost Downloads]# grep "password" /var/log/mysqld.log 
2018-05-14T03:32:58.673795Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: 9Moc6qd>xM=c
```

可以看到生成的临时密码是 `9Moc6qd>xM=c`

现在必须立刻修改密码，不然会报错：

```
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```


使用 `mysql -uroot -p` 然后输入临时密码登录，然后依照下面的命令修改密码

```
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');

# mysql 8 用
mysql> alter user 'root'@'localhost' identified by 'newpass';
```

!> 如果在此步报错 ERROR 1819，请向下翻查看原因及解决方法


### 开机启动

6.查看 mysqld 是否开机自启动，并设置为开机自启动：

```
chkconfig --list | grep mysqld
chkconfig mysqld on

# systemd 采用
systemctl is-enabled mysqld
systemctl enable mysqld
```

### 设置字符集

!> mysql 8 默认就是 `utf8mb4`

7.修改字符集为 UTF-8：


```
vim /etc/my.cnf
```

在 `[mysqld]` 部分添加：

```
character-set-server=utf8
```

在文件末尾新增 `[client]` 段，并在 `[client]` 段添加：

```
default-character-set=utf8
```


修改好之后重启 mysqld 服务：

```
service mysqld restart

# systemd 用
systemctl restart mysqld
```

查看修改结果：


```
mysql> show variables like "%character%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```


### 错误处理
注：在修改密码步骤，若设置的密码为简单密码，可能会出现如下错误：



```
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

这一错误其实与 `validate_password_policy` 值（或者其他密码策略）的设置有关：

`validate_password_policy` 值默认为 `1`，即 `MEDIUM`，所以刚开始设置的密码必须符合长度要求，且必须含有数字，小写或大写字母，特殊字符

如果我们只是做为测试用而不需要如此复杂的密码，可使用如下方式修改 `validate_password_policy` 值

!> 在 mysql8 下 `validate_password_policy` 变成 `validate_password.policy` 了

```
mysql> set global validate_password.policy=0;
Query OK, 0 rows affected (0.00 sec)
```

这样，对密码要求就只有长度了，而密码的最小长度由 `validate_password_length` 值决定

`validate_password_length` 参数默认为 `8`，它有最小值的限制，最小值为：

```
validate_password_number_count+ validate_password_special_char_count+ (2 * validate_password_mixed_case_count)
```

其中，`validate_password_number_count` 指定了密码中数字的长度，`validate_password_special_char_count` 指定了密码中特殊字符的长度， `validate_password_mixed_case_count` 指定了密码中大小字母的长度。这些参数的默认值均为1，所以 `validate_password_length` 最小值为 `4`，如果显性指定 `validate_password_length` 的值小于 `4`，尽管不会报错，但 `validate_password_length` 的值将设为 `4`


设置 `validate_password_length` 的值：

!> mysql 8 是 `validate_password.length`

```
mysql> set global validate_password.length=4;
Query OK, 0 rows affected (0.00 sec)
```

如果修改了 `validate_password_number_count`，`validate_password_special_char_count`，`validate_password_mixed_case_count` 中任何一个值，则 `validate_password_length` 将进行动态修改。



## ubuntu 安装 mysql 

安装：

``` bash
sudo apt update
sudo apt install mysql-server mysql-client
```

配置文件：
```
 /etc/mysql/mysql.conf.d/mysqld.cnf
```

测试：

``` bash
mysql -uroot -p
```

重启
``` bash
sudo service mysql restart
```

### 允许远程登陆

首先修改配置文件，允许所有 ip
```
bind-address            = 0.0.0.0

```

然后再 mysql shell 中输入如下命令：

``` bash
#grant all privileges on 库名.表名 to '用户名'@'IP地址' identified by '密码' with grant option;

grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
# 刷新
flush privileges;
```


## 参考

[CentOS6.5安装MySQL5.7详细教程](https://www.cnblogs.com/lzj0218/p/5724446.html)

## 图形话工具安装

ubuntu 18.04 安装 workbench

```
apt install mysql-workbench
```

https://blog.csdn.net/lhs960124/article/details/80404849

## SSH 连接

如果要通过 SSH 连接 ubuntu 上的 mysql, 需要安装 SSH server

https://blog.csdn.net/jackghq/article/details/54974141

``` bash
sudo apt install openssh-server
```

## 独立客户端

下载

https://dev.mysql.com/downloads/shell/

连接方式

https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-connection-using-parameters.html

例子

``` bash
shell> mysqlsh --uri user@localhost:33065 --user otheruser

shell> mysqlsh --mysqlx -u user -h localhost -P 33065

shell> mysqlsh --mysql -u user -h localhost
```