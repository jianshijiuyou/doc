# 切换默认 shell 为 zsh

现在第三方社区下载 z shell

然后在用户目录下的 .bash_profile 或者 .bash_login 或者 .profile 文件中加入以下代码保存退出重新登陆即完成。如果没有这几个文件则手动创建也给 .profile 文件就行

# pm2 使用

先在套件中心安装 node v8

然后再执行

``` bash
sudo npm install -g pm2
```

如果此时无法执行 pm2 请

配置 path，在 `~/.zshrc` 中最后加上

``` bash
export PATH=$PATH:/volume1/@appstore/Node.js_v8/usr/local/bin
```

?> 具体位置请注意自已安装时候的位置


由于群晖无法使用 `pm2 startup` ,所以需要手动处理下

在计划任务中添加一个触发任务，开机时执行，要用 root 用户才行，命令如下

``` bash
sudo -u username /volume1/@appstore/Node.js_v8/usr/local/bin/pm2 resurrect
```

?> username 请替换为平时使用时候的用户名

# 安装 ipkg 包管理

http://rescene.wikidot.com/synology-ipkg

以下方式仅适用 x86_64 架构

``` bash
wget http://ipkg.nslu2-linux.org/feeds/optware/syno-i686/cross/unstable/syno-i686-bootstrap_1.2-7_i686.xsh

chmod +x syno-i686-bootstrap_1.2-7_i686.xsh

sudo sh syno-i686-bootstrap_1.2-7_i686.xsh

# 此时已经安装完成， update
ipkg update
# 开始下载你想下载的吧
ipkg install programname
```

?> 如果不行重启下试试

此时你常用的用户应该还使用不了 ipkg, 请配入环境变量

``` bash
> tail ~/.zshrc
...
export PATH=$PATH:/opt/bin

> source ~/.zshrc
```