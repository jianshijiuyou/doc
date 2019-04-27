# 如何通过7个简单步骤保护您的Linux服务器

> [原文地址](https://medium.com/servers-101/how-to-secure-your-linux-server-6026cfcdefd8)

> 许多服务器时不时被黑客攻击。所以我决定写一个简短的教程，向您展示如何轻松保护您的 Linux 服务器。

这并不是一本全面的安全指南。

但是，它可以帮助您防止几乎90％的流行后端攻击，例如暴力登录尝试和 DDoS。

最好的部分是你可以在一两个小时内实现它们。

# 开始之前

1. 你需要一台Linux服务器。
2. 您需要对命令行有基本的了解。这是一张你可以使用的[备忘单](https://learncodethehardway.org/unix/bash_cheat_sheet.pdf)。

如果您已满足上述要求，请转到第一步。

# 1.配置SSH密钥

要访问远程服务器，您必须使用密码登录或使用 SSH 密钥。

密码的问题在于它们很容易暴力破解（您将在下面学习如何进一步防止这种情况）。此外，您还必须在需要访问服务器时随时键入它们。

要避免上述缺点，您必须设置 SSH 密钥身份验证。它比密码更安全，因为黑客不能暴力破解它们。

连接到服务器也更容易，更快，因为您不需要输入密码。

以下是如何为服务器设置SSH身份验证。

* 在本地计算机上，键入以下内容生成SSH密钥对：

    ``` bash
    ssh-keygen
    ```

    上面的命令将指导您完成几个步骤来生成SSH密钥。记下要存储密钥的文件。

* 使用以下命令将公钥添加到服务器：

    ``` bash
    ssh-copy-id username@remote_host
    ```

    请务必使用您的真实用户名和服务器的IP地址替换 username 和 remote_host 。系统将提示您输入密码。

* 尝试使用以下命令登录服务器：

    ``` bash
    ssh username@remote_host
    ```

    不要忘记将 username 和 remote_host 替换为服务器的详细信息。您应该注意到，这次不会提示您输入密码。

如果 ssh-copy-id 命令无法使用，请使用以下命令将公钥（这里是 id_rsa.pub）拷贝到服务器：

``` bash
scp -P port id_rsa.pub username@remote_host:~/.ssh
```

?> port 是端口号


然后在服务器执行

``` bash
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

# 2.保持系统时间最新

许多安全协议利用您的系统时间来运行 cron 作业，日期日志和执行其他关键任务。

如果您的系统时间不正确，可能会对您的服务器产生负面影响。为防止这种情况发生，您可以安装 NTP 客户端。此客户端将使您的系统时间与全球 NTP 服务器保持同步。

使用以下命令安装 NTP 客户端：

``` bash
sudo apt install ntp
```

您不再需要担心再次设置系统日期。

# 3.查看活动端口

服务器上的应用程序会公开某些端口，以便网络中的其他应用程序可以访问它们。

黑客还可以在您的服务器上安装后门，并公开一个端口，通过它可以控制服务器。

出于这个原因，我们不希望您的服务器在我们不知道的端口上侦听请求。

要查看活动端口，请使用以下命令：

``` bash
sudo ss -lntup
```

查看输出并调查您似乎并不熟悉的任何端口或进程。

尝试发现并追踪可能有害的服务和进程。

要开始前，请查看 [“bad” TCP/UDP ports](https://www.garykessler.net/library/bad_ports.html)。

# 4.设置防火墙

防火墙允许您停止/允许来自/来自服务器上特定端口的流量。为此，我通常使用 UFW（简单的防火墙）。

UFW 的工作原理是让您配置以下规则：

* 允许或否认
* 传入或传出的流量
* to or from
* 特定或所有端口

在本节中，您将阻止除明确允许的网络流量之外的所有网络流量。在安装其他程序时，请记住启用运行所需的必要端口。

## 设置 UFW

* Install ufw.

    ``` bash
    sudo apt-get install ufw
    ```

* 您可以拒绝所有传出流量...

    ``` bash
    sudo ufw default deny outgoing comment 'deny all outgoing traffic'
    ```

* ...或允许所有传出流量。

    ``` bash
    sudo ufw default allow outgoing comment 'allow all outgoing traffic'
    ```

* 接下来，我们要拒绝所有传入的流量......

    ``` bash
    sudo ufw default deny incoming comment 'deny all incoming traffic'
    ```

* ...除了SSH连接，以便我们可以访问系统。

    ``` bash
    sudo ufw limit in ssh comment 'allow SSH connections in'
    ```

* 如果您将UFW配置为拒绝所有传出流量，请不要忘记根据您的需要允许特定流量。以下是一些例子：

    ``` bash
    # 允许端口 53 上的流量 -- DNS
    sudo ufw allow out 53 comment 'allow DNS calls out'
    # 允许在 123 端口输出流量-- NTP
    sudo ufw allow out 123 comment 'allow NTP out'
    # 允许 HTTP，HTTPS 或 FTP 的流量输出 apt 可能需要这些，具体取决于您使用的是哪些来源
    sudo ufw allow out http comment 'allow HTTP traffic out'
    sudo ufw allow out https comment 'allow HTTPS traffic out'
    sudo ufw allow out ftp comment 'allow FTP traffic out'
    # allow whois
    sudo ufw allow out whois comment 'allow whois'
    # 允许端口 68 上的流量输出 -- the DHCP client
    # you only need this if you're using DHCP
    sudo ufw allow out 68 comment 'allow the DHCP client to update'
    ```

* 要拒绝端口 99 上的任何流量，请使用以下命令：

    ``` bash
    sudo ufw deny 99
    ```

* 最后，使用以下命令启动 UFW：

    ``` bash
    sudo ufw enable
    ```

* 您还可以使用以下命令查看 UFW 状态：

    ``` bash
    sudo ufw status
    ```

# 5.防止自动攻击

您可以使用两个实用程序来阻止大多数自动攻击：

* [PSAD](http://www.cipherdyne.org/psad/)
* [Fail2Ban](https://www.fail2ban.org/)

PSAD 和 Fail2Ban 之间的区别

我们了解到端口可以访问服务器上的应用程序。

攻击者可能决定扫描您的服务器以获取他们可能用于访问服务器的开放端口。

PSAD 监视网络活动以检测并可选地阻止此类扫描和其他类型的可疑流量，例如 DDoS 或 OS 指纹识别尝试。

另一方面，Fail2Ban 扫描各种应用程序（如 FTP ）的日志文件，并自动禁止显示恶意标志（如自动登录尝试）的 IP。

* [Install Fail2Ban.](https://zaiste.net/intro_fail2ban_with_ufw/)
* [Install PSAD.](https://gist.github.com/netson/c45b2dc4e835761fbccc)

# 6.安装 logwatch

服务器上的应用程序通常会将日志消息保存到日志文件中。除非您打算手动监视日志文件，否则需要安装 logwatch。

logwatch 扫描系统日志文件并对其进行汇总。

您可以直接从命令行运行它，也可以将其安排在定期计划中运行。例如，您可以配置 logwatch 以通过电子邮件向您发送日志文件的每日摘要。请注意，您的服务器需要[能够发送电子邮件](https://gist.github.com/adamstac/7462202)才能正常工作。

logwatch使用服务文件来了解如何读取和汇总日志文件。您可以在 `/usr/share/logwatch/scripts/services` 中查看所有库存服务文件。

logwatch的配置文件 `/usr/share/logwatch/default.conf/logwatch.conf` 指定默认选项。您可以通过命令行参数覆盖它们。

要在 Ubuntu 或 Debian 上安装 logwatch，请运行以下命令：

``` bash
apt-get install logwatch
```

对于其他 Linux 发行版的用户，请查看 Linode 的这篇[史诗指南](https://www.linode.com/docs/uptime/monitoring/monitor-systems-logwatch/)。

您可以尝试直接运行 logwatch，以防需要查看收集的样本。

``` bash
sudo /usr/sbin/logwatch --output stdout --format text --range yesterday --service all
```

最后，告诉 logwatch 每天发送一封电子邮件，其中包含我们日志文件的摘要。要执行此操作，请打开文件 `/etc/cron.daily/00logwatch` 并找到执行行，然后将其更改为以下内容：

``` bash
/usr/sbin/logwatch --output mail --format html --mailto root --range yesterday --service all
```

# 7.执行安全审核

保护 Linux 服务器后，您应该执行安全审核，以发现您可能错过的任何安全漏洞。

为此，您可以使用 Lynis，这是一个可以执行以下操作的开源软件：

* 安全审计。
* 一致性测试（例如PCI，HIPAA，SOx）。
* 渗透测试。
* 漏洞检测。
* 系统强化。

## How to use Lynis

首先，通过克隆他们的 Github 存储库来安装 Lynis。这可确保您安装最新版本的Lynis。

``` bash
git clone https://github.com/CISOfy/lynis
```

切换到我们克隆Lynis的目录：

``` bash
cd lynis
```

最后，使用下面的命令来运行你的第一次审核：

``` bash
./lynis audit system
```

您可以在他们的[官方网站](https://cisofy.com/lynis/)上了解有关 Lynis 的更多信息。

# 最后

欢迎您阅读有关加强 Linux 服务器的其他操作指南。我希望你能学到新东西。