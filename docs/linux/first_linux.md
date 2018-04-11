# 换源  

清华大学开源软件镜像站： [https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)  

# ubuntu 安装 mysql 

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

# 安装 virtualenv

``` bash
sudo apt install python-virtualenv
```

建立 python 虚拟环境
用 -p 参数指定 python 版本
``` bash
virtualenv -p python3 testenv
```

进入虚拟环境

```
source testenv/bin/activate
```

成功进入的标志
```
(testenv) jianxin@ubuntu:~/pycode$ 
```

# 配置 cnpm

node 请去官网下压缩包安装

cnpm 淘宝镜像：[http://npm.taobao.org/](http://npm.taobao.org/)


# 字体安装

首先下载字体，然后复制到 `/usr/share/fonts/` 目录下

然后依次执行以下命令

``` bash
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -fv
```

# python3.6 安装

下载源码包

复制到 opt 目录

添加配置

``` bash
./configure
```

也可以指定安装目录

```
./configure --prefix=/usr/bin/python3.6
```

编译源码

``` bash
sudo make
```

执行安装 

``` bash
sudo make install 
```

创建一个别名方便使用

修改 `～/.profile` 文件

```
alias py3="/xxxxxx/python3.6"
```

## 如果安装完成后 pip 无法安装模块，报 SSL 相关错误，请重新按照以下步骤安装

首先安装 openssl 和 libssl-dev(centos 安装 openssl-devel 还有 `yum install -y openssl-static`)

再重新配置安装

``` bash
./configure --with-ssl

sudo make

sudo make install
```


通过国内镜像安装模块

举个栗子
``` bash
pip install Django -i https://pypi.douban.com/simple
```

## 如果 python3.6 无法使用 sqlite3 ，请手动安装

首先安装 libsqlite3-dev和 sqlite3 (centos 上是 sqlite-devel)  

再重新配置安装 python3.6，方法同上

## 安装 mysqlclient

如果报错，请先安装 libmysqlclient-dev ：

``` bash
sudo apt install libmysqlclient-dev

# centos
yum install python-devel mysql-devel -y

```
然后再安装：

``` bash
pip install mysqlclient
```

##  使用 mysql 时报错 "Unknown system variable 'storage_engine'"

请将名称换成
``` bash
default_storage_engine
```

# centos 安装 gcc

``` bash
yum install -y gcc wget
yum groupinstall "Development tools"
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```