> [原文地址](https://blog.jing.do/Synology-DDNS-CloudNS)

1、将 godaddy 的 域名服务器换成 cloudns。记得将解析记录都记下来全部迁到 cloudns。

```
ns101.cloudns.net
ns102.cloudns.net
```

2、在 cloudns 中添加一个域名

创建好之后，增加一个A记录（注意如果你的NS是用的xxx.xxx.com，你只能创建abc.xxx.xxx.com），随便给的IP（当然最好是你现在私有云的IP），然后点击右侧小箭头

之后点击active，然后会给你一个url地址，尝试复制浏览器打开，应该提示OK，则设置成功。

登陆到synology私有云服务器，在“控制面板”打开“外部访问”，在“DDNS”中点击“自定义”，填写服务提供商CloudNS，query填入：

```
https://ipv4.cloudns.net/api/dynamicURL/?q=__PASSWORD__&ip=__MYIP__
```

保存之后，点击新增DDNS，填写如下信息：

* 服务提供商：选择刚增加的CloudNS
* 主机名：填写刚创建的域名abc.xxx.xxx.com
* 邮箱：写你的，会给你发邮件
* 密钥：还记得之前浏览器打开的链接吗，?q=密钥。填写进去


增加之后，会提示正常，使用浏览器登陆域名测试，大功告成！

由于DDNS使用的是动态IP，所以需要创建一个定时任务去更新，任务可以创建在任何服务器，当然推荐放在私有云上，控制面板->定时任务，增加脚本任务，跑一下代码（记得替换key）：

```
wget -q --read-timeout=0.0 --waitretry=5 --tries=400 --background https://ipv4.cloudns.net/api/dynamicURL/?q=generated_key
```