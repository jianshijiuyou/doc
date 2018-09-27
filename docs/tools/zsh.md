# 下载

https://ohmyz.sh/
https://github.com/robbyrussell/oh-my-zsh

# 字体

部分主题需要安装 [Powerline Fonts ](https://github.com/powerline/fonts)

首先下载字体，然后复制到 `/usr/share/fonts/` 目录下

然后依次执行以下命令

``` bash
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -fv
```

安装 zsh 的 oh-my-zsh 的 agnoster-zsh-theme 主题后分支图标乱码问题

需要设置终端字体为 `droid sans mono dotted for powerline regular`

![](http://os6ycxx7w.bkt.clouddn.com/images/d0908542-62ca-4d21-be21-95574bccc6ab.png)

字体安装成功的标志

``` shell
echo "\ue0b0 \u00b1 \ue0a0 \u27a6 \u2718 \u26a1 \u2699"
```

?> 不乱码

# 语法高亮

https://github.com/zsh-users/zsh-syntax-highlighting

配置好后 `.zshrc` 大概可能也许会多出下面一行（具体按照上面的官方配置来）

``` shell
source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

# 终端历史命令联想

https://github.com/zsh-users/zsh-autosuggestions

# 其他

去除终端中开头那些没用的用户名，计算机名

修改 `.zshrc` 文件，添加如下代码

``` shell
prompt_context() {}
```