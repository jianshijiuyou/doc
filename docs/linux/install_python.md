
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

## centos 安装 gcc

``` bash
yum install -y gcc wget
yum groupinstall "Development tools"
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```