## CentOS 安装 MySQL

> [原文连接](https://www.cnblogs.com/lzj0218/p/5724446.html)


1.检测系统是否已经安装过mysql或其依赖，若已装过要先将其删除，否则第 4 步使用 yum 安装时会报错：

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
```

!> 如果在此步报错ERROR 1819，请向下翻查看原因及解决方法


6.查看mysqld是否开机自启动，并设置为开机自启动：

```
chkconfig --list | grep mysqld
chkconfig mysqld on
```

7.修改字符集为UTF-8：


```
vim /etc/my.cnf
```

在[mysqld]部分添加：

```
character-set-server=utf8
```

在文件末尾新增[client]段，并在[client]段添加：

```
default-character-set=utf8
```


修改好之后重启mysqld服务：

```
service mysqld restart
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

这一错误其实与 `validate_password_policy` 值的设置有关：

`validate_password_policy` 值默认为 `1`，即 `MEDIUM`，所以刚开始设置的密码必须符合长度要求，且必须含有数字，小写或大写字母，特殊字符

如果我们只是做为测试用而不需要如此复杂的密码，可使用如下方式修改 `validate_password_policy` 值

!> 在 mysql8 下 `validate_password_policy` 变成 `validate_password.policy` 了

```
mysql> set global validate_password_policy=0;
Query OK, 0 rows affected (0.00 sec)
```

这样，对密码要求就只有长度了，而密码的最小长度由validate_password_length值决定

validate_password_length参数默认为8，它有最小值的限制，最小值为：

```
validate_password_number_count+ validate_password_special_char_count+ (2 * validate_password_mixed_case_count)
```

其中，validate_password_number_count指定了密码中数字的长度，validate_password_special_char_count指定了密码中特殊字符的长度，validate_password_mixed_case_count指定了密码中大小字母的长度。这些参数的默认值均为1，所以validate_password_length最小值为4，如果显性指定validate_password_length的值小于4，尽管不会报错，但validate_password_length的值将设为4


设置validate_password_length的值：

```
mysql> set global validate_password_length=4;
Query OK, 0 rows affected (0.00 sec)
```

如果修改了validate_password_number_count，validate_password_special_char_count，validate_password_mixed_case_count中任何一个值，则validate_password_length将进行动态修改。



## ubuntu 安装 mysql 

安装：

``` bash
sudo apt install mysql-server
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

允许远程登陆：

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