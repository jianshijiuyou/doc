## 换源  

清华大学开源软件镜像站： [https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)  


## 配置 cnpm

node 请去官网下压缩包安装

cnpm 淘宝镜像：[http://npm.taobao.org/](http://npm.taobao.org/)

## 常用的 RPM 软件包命令

| 说明      |     命令 |
| :-------- | :--------|
|安装软件的命令格式	|rpm -ivh filename.rpm
|升级软件的命令格式|	rpm -Uvh filename.rpm
|卸载软件的命令格式|	rpm -e filename.rpm
|查询软件描述信息的命令格式	|rpm -qpi filename.rpm
|列出软件文件信息的命令格式	|rpm -qpl filename.rpm
|查询文件属于哪个RPM的命令格式|	rpm -qf filename
|查询软件的安装目录信息|	例：`rpm -ql nginx`


## 常见的 Yum 命令

| 命令      |     作用 | 
| :-------- | :--------|
|yum repolist all	|列出所有仓库
|yum list all	|列出仓库中所有软件包
|yum list &#124; grep xxx 	|查找指定软件包 xxx
|yum info 软件包名称	|查看软件包信息
|yum install 软件包名称	|安装软件包
|yum reinstall 软件包名称	|重新安装软件包
|yum update 软件包名称	|升级软件包
|yum remove 软件包名称	|移除软件包
|yum clean all	|清除所有仓库缓存
|yum check-update	|检查可更新的软件包
|yum grouplist	|查看系统中已经安装的软件包组
|yum groupinstall 软件包组	|安装指定的软件包组
|yum groupremove 软件包组|	移除指定的软件包组
|yum groupinfo 软件包组|	查询指定的软件包组信息

## systemd 与 System V init 的区别以及作用

| System V init 运行级别      |     systemd 目标名称 |   作用   |
| :-------- | :--------| :------ |
|0 | runlevel0.target, poweroff.target  |关机
|1 | runlevel1.target, rescue.target  |单用户模式
|2|  runlevel2.target, multi-user.target | 等同于级别 3
|3 | runlevel3.target, multi-user.target | 多用户的文本界面
|4 | runlevel4.target, multi-user.target | 等同于级别 3
|5 | runlevel5.target, graphical.target  |多用户的图形界面
|6 | runlevel6.target, reboot.target  |重启
|emergency | emergency.target  |紧急 Shell


如果想要将系统默认的运行目标修改为“多用户，无图形”模式，可直接用 ln 命令把多用户模式目标文件连接到 `/etc/systemd/system/` 目录：

```
[root@linuxprobe ~]# ln -sf /lib/systemd/system/multi-user.target /etc/systemd/system/default.target
```

## systemctl 管理服务的启动、重启、停止、重载、查看状态等常用命令

|System V init命令（RHEL 6系统）	|systemctl命令（RHEL 7系统）|	作用 |
| :-------- | :--------| :------ |
|service foo start	|systemctl start foo.service	|启动服务
|service  foo restart	|systemctl restart foo.service|	重启服务
|service foo stop|	systemctl stop foo.service	|停止服务
|service foo reload|	systemctl reload foo.service|	重新加载配置文件（不终止服务）
|service foo status	|systemctl status foo.service	|查看服务状态


## systemctl 设置服务开机启动、不启动、查看各级别下服务启动状态等常用命令

|System V init命令（RHEL 6系统）|	systemctl命令（RHEL 7系统）	|作用
| :-------- | :--------| :------ |
|chkconfig foo on|	systemctl enable foo.service|	开机自动启动
|chkconfig foo off	|systemctl disable foo.service|	开机不自动启动
|chkconfig foo	|systemctl is-enabled foo.service	|查看特定服务是否为开机自动启动
|chkconfig --list	|systemctl list-unit-files --type=service	|查看各个级别下服务的启动与禁用情况



## 常用系统工作命令

### echo

echo 命令用于在终端输出字符串或变量提取后的值，格式为 “echo [字符串 | $变量]”。

``` bash
[jiuyou@localhost ~]$ echo hello,jack
hello,jack
[jiuyou@localhost ~]$ echo $SHELL
/bin/bash
```

### date

date 命令用于显示及设置系统的时间或日期，格式为“date [选项] [+指定的格式]”。

只需在强大的 date 命令中输入以“+”号开头的参数，即可按照指定格式来输出系统的时间或日期。

|参数|	作用
|:------|:--------:
|%t	|跳格[Tab键]
|%H	|小时（00～23）
|%I	|小时（00～12）
|%M	|分钟（00～59）
|%S	|秒（00～59）
|%j	|今年中的第几天

按照默认格式查看当前系统时间的 date 命令如下所示：

``` bash
[root@linuxprobe ~]# date
Mon Aug 24 16:11:23 CST 2017
```

按照“年-月-日 小时:分钟:秒”的格式查看当前系统时间的 date 命令如下所示：

```
[root@linuxprobe ~]# date "+%Y-%m-%d %H:%M:%S"
2017-08-24 16:29:12
```
将系统的当前时间设置为 2017 年 9 月 1 日 8 点 30 分的 date 命令如下所示：

```
[root@linuxprobe ~]# date -s "20170901 8:30:00"
Fri Sep 1 08:30:00 CST 2017
```

date 命令中的参数%j 可用来查看今天是当年中的第几天。这个参数能够很好地区分备份时间的新旧，即数字越大，越靠近当前时间。该参数的使用方式以及显示结果如下所示。

```
u@localhost ~]$ date +%j
103
```

### reboot

reboot 命令用于重启系统，其格式为 reboot。

```
[root@linuxprobe ~]# reboot
```

### poweroff

poweroff 命令用于关闭系统，其格式为 poweroff。

```
[root@linuxprobe ~]# poweroff
```

### ln

创建链接（可以理解成快捷方式）

`ln 文件 链接名` 默认创建硬链接

`ln -s 文件 链接名` 软连接

!> 一定要用绝对路径，否则链接无法使用

### ps

ps 命令用于查看系统中的进程状态。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/fdf2a8dc-f42c-43cc-9a04-1cf01c4b0ec1.png)

Linux 系统中时刻运行着许多进程，如果能够合理地管理它们，则可以优化系统的性能。在 Linux 系统中，有 5 种常见的进程状态，分别为运行、中断、不可中断、僵死与停止，其各自含义如下所示。
 * **R（运行）**：进程正在运行或在运行队列中等待。
 * **S（中断）**：进程处于休眠中，当某个条件形成后或者接收到信号时，则脱离该   状态。
 * **D（不可中断）**：进程不响应系统异步信号，即便用kill命令也不能将其中断。
 * **Z（僵死）**：进程已经终止，但进程描述符依然存在, 直到父进程调用wait4()系统函数后将进程释放。
 * **T（停止）**：进程收到停止信号后停止运行。

当执行 `ps aux` 命令后通常会看到如下进程状态

> 或者 `ps -ef`

|USER|	PID|	%CPU|	%MEM|	VSZ|	RSS|	TTY|	STAT	|START|	TIME|	COMMAND
|:----|:----|:-------|:------|:-----|:------|:-------|:----------|:---|:-------|:---------------------------
|进程的所有者	|进程ID号|	运算器占用率|	内存占用率|	虚拟内存使用量（单位是KB）|	占用的固定内存量（单位是KB）|	所在终端	|进程状态|	被启动的时间|	实际使用CPU的时间	|命令名称与参数
|root|	1|	0.0|	0.4|	53684| 	7628|	?|	Ss|	07 :22|	0:02|	/usr/lib/systemd/systemd
|root|	2|	0.0|	0.0|	0|	0|	?|	S|	07:22|	0:00|	[kthreadd]
|root|	3|	0.0|	0.0|	0|	0|	?|	S|	07:22|	0:00|	[ksoftirqd/0]
|root|	5|	0.0|	0.0|	0|	0|	?|	S<|	07:22|	0:00|	[kworker/0:0H]
|root|	7|	0.0|	0.0|	0|	0|	?|	S	|07:22|	0:00|	[migration/0]

### top 
top 命令用于动态地监视进程活动与系统负载等信息，其格式为 top。

top 命令执行结果的前 5 行为系统整体的统计信息，其所代表的含义如下。
 * 第 1 行：系统时间、运行时间、登录终端数、系统负载（三个数值分别为 1 分钟、5分钟、15 分钟内的平均值，数值越小意味着负载越低）。
 * 第 2 行：进程总数、运行中的进程数、睡眠中的进程数、停止的进程数、僵死的进程数。
 * 第 3 行：用户占用资源百分比、系统内核占用资源百分比、改变过优先级的进程资源百分比、空闲的资源百分比等。
 * 第 4 行：物理内存总量、内存使用量、内存空闲量、作为内核缓存的内存量。
 * 第 5 行：虚拟内存总量、虚拟内存使用量、虚拟内存空闲量、已被提前加载的内存量。


### pidof
pidof 命令用于查询某个指定服务进程的 PID 值，格式为“pidof [参数] [服务名称]”。

```
[root@linuxprobe ~]# pidof sshd
2156
```

### which

显示出某个命令的实际位置

```
[root@localhost ~]# which vim
/bin/vim
[root@localhost ~]# which rm
alias rm='rm -i'
	/bin/rm
[root@localhost ~]# 
```


### kill

http://linux.vbird.org/linux_basic/0440processcontrol.php#process_2

kill 命令用于终止某个指定 PID 的服务进程，格式为 `kill -signal %jobnumber`。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/3bcf2325-48b7-43ee-8ab3-c212157bd788.png)


### killall
killall 命令用于终止某个指定名称的服务所对应的全部进程，格式为：“killall [参数] [进程名称]”。

```
[root@linuxprobe ~]# pidof httpd
13581 13580 13579 13578 13577 13576
[root@linuxprobe ~]# killall httpd
[root@linuxprobe ~]# pidof httpd
[root@linuxprobe ~]#
```

## 系统状态检测命令

### 系统版本内核CPU
uname 命令用于查看系统内核与系统版本等信息，格式为“uname [-a]”。

或者 `cat /proc/version`

```
[jiuyou@localhost ~]$ uname -a 
Linux localhost.localdomain 3.10.0-693.el7.x86_64 #1 SMP Tue Aug 22 21:09:27 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

顺带一提，如果要查看当前系统版本的详细信息，则需要查看 redhat-release 文件（或 os-release 文件）
```
[jiuyou@localhost ~]$ cat /etc/redhat-release 
CentOS Linux release 7.4.1708 (Core) 
```

或

```
[root@6eaaa85720e0 /]# cat /etc/os-release 
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

```

查看 CPU 信息

```
cat /proc/cpuinfo
```

### 磁盘 

#### df

查看各分区使用情况：

```
df -h
```

#### du

查看当前目录下 **文件** 及 **文件夹** 大小：

```
du -sh *

# 当前文件夹大小
du -sh
```

查看当前目录下所有 **文件夹（包括隐藏文件夹）** 大小：

```
du -h -d 1
```
`-d` or `--max-depth=N` 指定文件夹深度


### uptime
uptime 用于查看系统的负载信息，格式为 uptime。

它可以显示当前系统时间、系统已运行时间、启用终端数量以及平均负载值等信息。平均负载值指的是系统在最近 1 分钟、5 分钟、15 分钟内的压力情况（下面加粗的信息部分）；负载值越低越好，尽量不要长期超过 1，在生产环境中不要超过 5。

```
[jiuyou@localhost ~]$ uptime
 23:49:15 up  3:33,  2 users,  load average: 0.16, 0.11, 0.07
```

### free 
free 用于显示当前系统中内存的使用量信息，格式为“free [-h]”。

查看内存及用量：

```
free -m  # M 为单位

free -g # G 为单位
```

执行 `free -h`

|-|内存总量	|已用量|	可用量	|进程共享的内存量	|磁盘缓存的内存量|	缓存的内存量
|:---|:----|:----|:---|:----|:-----|:---------------
|	|total	|used	|free|	shared	|buffers	|cached
|Mem|	1.8GB|	1.3GB	|542MB	|9.8MB	|1.6MB|	413MB
|-/+ buffers/cache|	|	869MB|	957MB	|		
|Swap|	2.0GB|	0|	2.0GB		|	

### who

who 用于查看当前登入主机的用户终端信息，格式为“who [参数]”。

### last

last 命令用于查看所有系统的登录记录，格式为“last [参数]”。

### history
history 命令用于显示历史执行过的命令，格式为“history [-c]”。

执行 history 命令能显示出当前用户在本地计算机中执行过的最近 1000 条命令记录。如果觉得 1000 不够用，还可以自定义/etc/profile 文件中的 HISTSIZE 变量值。

-c 参数会清空所有的命令历史记录。

历史命令会被保存到用户家目录中的 .bash_history 文件中。

### sosreport

sosreport 命令用于收集系统配置及架构信息并输出诊断文档，格式为 sosreport。

## 网络相关

### wget 
wget 命令用于在终端中下载网络文件，格式为“wget [参数] 下载地址”。

|参数	|作用
|:---:|:----:|
|-b	|后台下载模式
|-P	|下载到指定目录
|-t	|最大尝试次数
|-c	|断点续传
|-p	|下载页面内所有资源，包括图片、视频等
|-r	|递归下载

```
[root@linuxprobe ~]# wget http://www.linuxprobe.com/docs/LinuxProbe.pdf
```

### curl
curl 命令是一个利用 URL 规则在命令行下工作的文件传输工具。它支持文件的上传和下载，所以是综合传输工具，但按传统，习惯称 curl 为下载工具。作为一款强力工具，curl 支持包括 HTTP、HTTPS、ftp 等众多协议，还支持 POST、cookies、认证、从指定偏移处下载部分文件、用户代理字符串、限速、文件大小、进度条等特征。做网页处理流程和数据检索自动化，curl 都可以祝你一臂之力。

语法： `curl (选项) (参数)`

例：用 curl 请求百度的首页
```
curl www.baidu.com
```
百度的首页 html 会被输出到 stdout


选项 `-o` 将下载数据写入到指定名称的文件中，并使用 `--progress` 显示进度条：
```
curl www.baidu.com -o baidu.html --progress
```

使用 `-H "头部信息"` 传递多个头部信息，例如：
```
curl -H "Host:man.linuxde.net" -H "accept-language:zh-cn" URL
```

使用 curl 选项 -u 可以完成HTTP或者FTP的认证，可以指定密码，也可以不指定密码在后续操作中输入密码：

```
curl -u user:pwd http://man.linuxde.net

# or

curl -u user http://man.linuxde.net
```

通过 `-I` 或者 `-head` 可以只打印出HTTP头部信息：

```
[root@6eaaa85720e0 ~]# curl -I www.baidu.com                         
HTTP/1.1 200 OK
Server: bfe/1.0.8.18
Date: Mon, 07 May 2018 14:46:01 GMT
Content-Type: text/html
Content-Length: 277
Last-Modified: Mon, 13 Jun 2016 02:50:35 GMT
Connection: Keep-Alive
ETag: "575e1f7b-115"
Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
Pragma: no-cache
Accept-Ranges: bytes

```

显示请求头和响应头信息，但不要响应内容

```
curl -v www.baidu.com > /dev/null
```

### netstat
> https://linux.cn/article-2434-1.html

用于列出系统上所有的网络套接字连接情况，包括 tcp, udp 以及 unix 套接字。另外它还能列出处于监听状态（即等待接入请求）的套接字。如果你想确认系统上的 Web 服务有没有起来，你可以查看 80 端口有没有打开。

| 选项 | 作用 |
|:----- | :--- |
| `-a` | 列出所有当前的连接 |
| `-t` | 只列出 TCP 协议的连接。 <br/>例：`netstat -at` |
| `-u` | 只列出 UDP 协议的连接。 <br/>例：`netstat -au` |
| `-n` | 禁用域名解析功能。 <br/>例：`netstat -aun` |
| `-l` | 列出正在监听的套接字。 <br/>例：`netstat -tnl` |
| `-p` | 查看进程信息。（查看 root 用户的进程要 root 权限） <br/>例：`sudo netstat -ltpn` |
| `-ep` | 同时查看进程名和用户名。 <br/>例：`sudo netstat -ltep` |

### iproute2

iproute2 工具集用于替代 net-tools（2001 年就停止维护了） 工具集

https://my.oschina.net/lionel45/blog/109779

https://linux.cn/article-4326-1.html

| 用途 |	net-tools（被淘汰）	| iproute2
|:--------|:-------------------|:----------
| 地址和链路配置|	ifconfig	|ip addr, ip link
| 路由表|	route	|ip route
| 邻居	|arp	|ip neigh
| VLAN	|vconfig	|ip link
| 隧道	|iptunnel	|ip tunnel
| 组播	|ipmaddr	|ip maddr
| 统计	|netstat	|ss

常用组合

| 命令 |	说明
|:--------|:-------------------
| `ip addr` | 等价 `ifconfig`	
| `ss -s` | 查看当前服务器的网络连接统计
| `ss -l` | 查看所有打开的网络端口
| `ss -pl` | 查看所有打开的网络端口, 列出具体的程序名称
| `ss -a` | 查看所有的 socket 连接
| `ss -ta` | 查看 TCP sockets，使用-ta选项
| `ss -tan` | 屏蔽 DNS 解析，使用 -n
| `ss -ua` | 查看UDP sockets，使用-ua选项
| `ss -wa` | 查看RAW sockets，使用-wa选项
| `ss -xa` | 查看UNIX sockets，使用-xa选项
| `ss -ltnp` | 等价 `netstat -ltnp`

![](http://pikachu666.oss-cn-hongkong.aliyuncs.com/images/7ee77852-3680-4b20-a907-408965d00b82.png)

ip 配置

http://linux.vbird.org/linux_server/0140networkcommand.php#ip_cmd

```
$ sudo ip addr add 10.0.0.1/24 dev eth1
$ sudo ip link set up eth1
```

网络相关配置文件案

http://cn.linux.vbird.org/linux_server/0130internet_connect_1.php#note_files

### ifconfig

ifconfig 命令用于获取网卡配置与网络状态等信息，格式为“ifconfig [网络设备] [参数]”。

### nmap

一个强大的网络检测工具

https://blog.gtwang.org/linux/nmap-command-examples-tutorials/

安装

```
sudo yum install nmap
# or
sudo apt install nmap
```

基本用法，检查目标端口

```
nmap www.baidu.com
# or
nmap 127.0.0.1

# 详细信息
nmap -v 127.0.0.1

# 多主机扫描
nmap www.hinet.net tw.yahoo.com www.google.com.tw

# 扫描整个子网
nmap 192.168.1.*
# or
nmap 192.168.1.133/24

# 扫描子网内的某几台主机
nmap 192.168.0.123,124,125

# 扫描子网内的某一段
nmap 192.168.0.123-140
```

检测各种服务的版本信息

```
nmap -A 127.0.0.1
```

## 工作目录切换命令

### pwd
pwd 命令用于显示用户当前所处的工作目录，格式为“pwd [选项]”。

### cd
cd 命令用于切换工作路径，格式为“cd [目录名称]”。

### ls

ls 命令用于显示目录中的文件信息，格式为“ls [选项] [文件] ”。

| 参数      |     意义 |   
| :-------- | :--------|
| -a    |   查看全部文件（包括隐藏文件） | 
| -l |  查看文件的属性、大小等详细信息 |
| -h |  size 自动换算单位|
| -d | 查看目录属性信息  例：`ls -ld /etc` |

## 文本文件编辑命令

### cat
cat 命令用于查看纯文本文件（内容较少的），格式为“cat [选项] [文件]”。

| 参数      |     意义 |   
| :-------- | :--------|
| -n    |   显示行号 | 

### more
more 命令用于查看纯文本文件（内容较多的），格式为“more [选项]文件”。

more 命令会在最下面使用百分比的形式来提示您已经阅读了多少内容。您还可以使用空格键或回车键向下翻页：

### head

head 命令用于查看纯文本文档的前 N 行，格式为“head [选项] [文件]”。

查看文本中前 20 行的内容

```
head -n 20 xxxx.txt
```

### tail

tail 命令用于查看纯文本文档的后 N 行或持续刷新内容，格式为“tail [选项] [文件]”。

查看文本内容的最后 20 行: `tail -n 20`

持续刷新一个文件的内容，当想要实时查看最新日志文件时，这特别有用，此时的命令格式为 “tail -f 文件名”

### tr

tr 命令用于替换文本文件中的字符，格式为“tr [原始字符] [目标字符]”。

把某个文本内容中的英文全部替换为大写：

```
cat anaconda-ks.cfg | tr [a-z] [A-Z]
```

### wc
wc 命令用于统计指定文本的行数、字数、字节数，格式为“wc [参数] 文本”。

|参数|	作用
|:--|:---
|-l	|只显示行数
|-w	|只显示单词数
|-c	|只显示字节数

统计当前系统中有多少个用户

```
[root@linuxprobe ~]# wc -l /etc/passwd
38 /etc/passwd
```

### stat
stat 命令用于查看文件的具体存储信息和时间等信息，格式为“stat 文件名称”。

### cut
cut 命令用于按“列”提取文本字符，格式为“cut [参数] 文本”。

使用-f 参数来设置需要看的列数，还需要使用-d 参数来设置间隔符号。passwd 在保存用户数据信息时，用户信息的每一项值之间是采用冒号来间隔的，接下来我们使用下述命令尝试提取出 passwd 文件中的用户名信息，即提取以冒号（：）为间隔符号的第一列内容：

```
[root@linuxprobe ~]# head -n 2 /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
[root@linuxprobe ~]# cut -d: -f1 /etc/passwd
root
bin
daemon
adm
lp
sync
...
```

### diff
diff 命令用于比较多个文本文件的差异，格式为“diff [参数] 文件”。

在使用 diff 命令时，不仅可以使用--brief 参数来确认两个文件是否不同，还可以使用-c 参数来详细比较出多个文件的差异之处，这绝对是判断文件是否被篡改的有力神器。


## 文件目录管理命令

### touch
touch 命令用于创建空白文件或设置文件的时间，格式为“touch [选项] [文件]”。

### mkdir 
mkdir 命令用于创建空白的目录，格式为“mkdir [选项] 目录”。

结合-p 参数来递归创建出具有嵌套叠层关系的文件目录。

### cp

cp 命令用于复制文件或目录，格式为“cp [选项] 源文件 目标文件”。

在 Linux 系统中，复制操作具体分为 3 种情况：
 * 如果目标文件是目录，则会把源文件复制到该目录中；
 * 如果目标文件也是普通文件，则会询问是否要覆盖它；
 * 如果目标文件不存在，则执行正常的复制操作。


|参数	|作用|
|:--|:----
|-p|	保留原始文件的属性
|-d	|若对象为“链接文件”，则保留该“链接文件”的属性
|-r|	递归持续复制（用于目录）
|-i	|若目标文件存在则询问是否覆盖
|-a|	相当于-pdr（p、d、r为上述参数）

### mv

mv 命令用于剪切文件或将文件重命名，格式为“mv [选项] 源文件 [目标路径|目标文件名]”。

### rm

rm 命令用于删除文件或目录，格式为“rm [选项] 文件”。

系统会默认向您询问是否要执行删除操作，如果不想总是看到这种反复的确认信息，可在 rm 命令后跟上-f 参数来强制删除。另外，想要删除一个目录，需要在 rm 命令后面一个-r 参数才可以，否则删除不掉。

### dd

dd 命令用于按照指定大小和个数的数据块来复制文件或转换文件，格式为“dd [参数]”。

|参数	|作用
|:----|:--
|if	|输入的文件名称
|of	|输出的文件名称
|bs	|设置每个“块”的大小
|count	|设置要复制“块”的个数

Linux系统中有一个名为/dev/zero 的设备文件，每次在课堂上解释它时都充满哲学理论的色彩。因为这个文件不会占用系统存储空间，但却可以提供无穷无尽的数据，因此可以使用它作为 dd 命令的输入文件，来生成一个指定大小的文件。

例如我们可以用 dd 命令从/dev/zero 设备文件中取出一个大小为 560MB 的数据块，然后保存成名为 560_file 的文件。在理解了这个命令后，以后就能随意创建任意大小的文件了：

```
[root@linuxprobe ~]# dd if=/dev/zero of=560_file count=1 bs=560M
1+0 records in
1+0 records out
587202560 bytes (587 MB) copied, 27.1755 s, 21.6 MB/s
```

dd 命令的功能也绝不仅限于复制文件这么简单。如果您想把光驱设备中的光盘制作成 iso 格式的镜像文件，在 Windows 系统中需要借助于第三方软件才能做到，但在 Linux 系统中可以直接使用 dd 命令来压制出光盘镜像文件，将它变成一个可立即使用的 iso 镜像：

```
[root@linuxprobe ~]# dd if=/dev/cdrom of=RHEL-server-7.0-x86_64-LinuxProbe.Com.iso
7311360+0 records in
7311360+0 records out
3743416320 bytes (3.7 GB) copied, 370.758 s, 10.1 MB/s
```

### file
file 命令用于查看文件的类型，格式为“file 文件名”。

```
[root@linuxprobe ~]# file anaconda-ks.cfg
anaconda-ks.cfg: ASCII text
[root@linuxprobe ~]# file /dev/sda
/dev/sda: block special
```

## 打包压缩

| 命令 | 说明
|:-----|:------
| xz | 使用 LZMA 算法的高性能解压缩工具
| gzip | 流行的 GNU gzip 解压缩程序 <br> 压缩：`gzip filename` <br> 解压缩：`gzip -d filename.gz`
| bzip2 | 免费无专利 <br> 压缩：`bzip2 filename` <br> 解压缩：`bzip2 -d filename.bz2`
| zip/unzip | 兼容 WinZIP
| rar | 兼容 WinRAR
| 7za | 使用 LZMA 算法的高性能解压缩工具
| tar | 文件打包、归档 <br> 压缩：`tar -czvf 压缩包名称.tar.gz 要打包的目录` <br> 解压缩：`tar -xzvf 压缩包名称.tar.gz`

| 文件后缀 | 说明 |
|:-----|:----
| .bz2 | bzip2
| .gz | gzip
| .xz | xz 
| .tar | tar
| .tbz | tar + bzip2
| .tgz | tar + gzip
| .zip | zip
| .rar | rar
| .7z | 7za

tar 命令常用参数解释，格式为“tar [选项] [文件]”。

|参数	|作用
|:---|:--
|-c	|创建压缩文件
|-x	|解开压缩文件
|-t	|查看压缩包内有哪些文件
|-z	|用Gzip压缩或解压
|-j	|用bzip2压缩或解压
|-v	|显示压缩或解压的过程
|-f	|目标文件名
|-p	|保留原始的权限与属性
|-P	|使用绝对路径来压缩
|-C	|指定解压到的目录

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/20180628205407.png)

## 搜索

### grep

grep 命令用于在文本中执行关键词搜索，并显示匹配的结果，格式为“grep [选项] [文件]”。

|参数|	作用
|:-----|:------
|-b|	将可执行文件（binary）当作文本文件（text）来搜索
|-c	|仅显示找到的行数
|-i	|忽略大小写
|-n	|显示行号
|-v	|反向选择—仅列出没有“关键词”的行

一些例子

在文件 myfile 中查找包含字符串 mystr 的行
```
grep -n mystr myfile
```

显示 myfile 中第一个字符为字母的所有行

```
grep '^[a-zA-Z]' myfile
```

在文件 myfile 中查找首字符不是 # 的行（即过滤掉注释行）

```
grep -v '^#' myfile
```

列出 `/etc` 目录（包括子目录）下所有文件内容中包含字符串 “root” 的文件名

```
grep -lr root /etc/*
```

查找 `/etc/passwd` 文件中以 `a` 开头的行

```
grep ^a /etc/passwd
```
查找 `/etc/passwd` 文件中以 `a` 开头的行，显示行号

```
grep -n ^a /etc/passwd
```

查找/etc/passwd文件中以login结束的行；

```
grep login$ /etc/passwd
```



### find

find 命令用于按照指定条件来查找文件，格式为“find [查找路径] 寻找条件 操作”。

|参数	|作用
|:--|:---
|-name	|匹配名称
|-perm|	匹配权限（mode为完全匹配，-mode为包含即可）
|-user |	匹配所有者
|-group	|匹配所有组
|-mtime -n +n	|匹配修改内容的时间（-n指n天以内，+n指n天以前）
|-atime -n +n	|匹配访问文件的时间（-n指n天以内，+n指n天以前）
|-ctime -n +n	|匹配修改文件权限的时间（-n指n天以内，+n指n天以前）
|-nouser	|匹配无所有者的文件
|-nogroup	|匹配无所有组的文件
|-newer f1 !f2	|匹配比文件f1新但比f2旧的文件
|--type b/d/c/p/l/f	|匹配文件类型（后面的字母参数依次表示块设备、目录、字符设备、管道、链接文件、文本文件）
|-size	|匹配文件的大小（+50KB为查找超过50KB的文件，而-50KB为查找小于50KB的文件）
|-prune	|忽略某个目录
|-exec …… {}\;	|后面可跟用于进一步处理搜索结果的命令（下文会有演示）



根据文件系统层次标准（Filesystem Hierarchy Standard）协议，Linux 系统中的配置文件会保存到/etc 目录中（详见第 6 章）。如果要想获取到该目录中所有以 host 开头的文件列表，可以执行如下命令：

```
[root@linuxprobe ~]# find /etc -name "host*" -print
/etc/avahi/hosts
/etc/host.conf
/etc/hosts
/etc/hosts.allow
/etc/hosts.deny
/etc/selinux/targeted/modules/active/modules/hostname.pp
/etc/hostname
```

如果要在整个系统中搜索权限中包括 SUID 权限的所有文件（详见第 5 章），只需使用-4000 即可：
```
[root@linuxprobe ~]# find / -perm -4000 -print
/usr/bin/fusermount
/usr/bin/su
/usr/bin/umount
/usr/bin/passwd
/usr/sbin/userhelper
/usr/sbin/usernetctl
...
```

## 其他

### 代码行数查看

例子

```
find ./src -name "*.py" |xargs cat|grep -v ^$|wc -l
```