> 摘要自：[https://juejin.im/book/5a124b29f265da431d3c472e](https://juejin.im/book/5a124b29f265da431d3c472e)


# 常用 Git 命令
| 命令 | 说明
|:-----|:------
| `git clone <URL>` | 获取远程仓库的文件到本地
| `git init` | 在本地初始化仓库
| `git remote add <origin> <URL>` | 添加远程仓库
| `git status` | 查看工作目录当前状态
| `git diff` | 查看当前工作目录的改动
| `git add <filename>` | 添加要跟踪的文件 <br> 常用: `git add .`
| `git commit [-m "log"]` | 提交，`-m` 后跟提交时附带的日志
| `git log` | 查看提交历史 <br> 查看详细历史：`git log -p` <br> 简要统计：`git log --stat`
| `git show [commit id]` | 查看 commit 的具体改动，默认查看当前 commit
| `git push [仓库名] [分支名]` | 把本地的提交记录推送到服务器 <br> 常用: `git push -u origin master` <br> 不指定默认推送 `master` 分支 <br> 有时需要指定其他分支: `git push origin newBranch`
| `git pull` | 把服务器提交记录拉取到本地
| `git branch <branch name>` | 创建分支
| `git checkout <branch name>` | 切换分支
| `git checkout -b <branch name>` | 创建分支同时切换到该分支上
| `git checkout .` | 放弃当前所有没有提交的更改
| `git branch -d <branch name>` | 删除分支，<br>执行完后再执行 `git push origin -d <branch name>`, <br>可以把服务器端的分支也删除了
| `git branch -a (--all)` | 列出远程跟踪分支和本地分支。
| `git merge <branch name>` | 把指定分支合并到当前分支上 <br> 一般在 master 分支上操作，合并其他分支
| `git merge --abort` | 取消合并
| `git rebase master` | 把当前分支所有的 commit 提交到 master 分支上 <br> 一般执行还需要执行 `git checkout master` 和 `git merge branch1` <br> **一般不会在 master 分支上执行 `rebase`**
| `git config --global core.editor vim` | 将默认 commit 的编辑器修改为 vim |
| `git config –global user.name "YOUR NAME"` | 全局配置用户名
| `git config –global user.email "YOUR EMAIL ADDRESS"` | 全局配置邮箱

# 问题修复

## 修正最新的提交

`--amend` 选项可以覆盖最新的 commit，所以可以用来修正错误
``` 
git add 修改后的文件
git commit --amend
```

## 修正非最新的提交

用 `rebase -i`

## 丢弃最新的提交

`reset` 指令可以重置 `HEAD` 和 `branch` 的位置，达到丢弃 commit 效果
```
git reset --hard HEAD^
```
!> 这会清除当前的改动。不管它们是否被放进暂存区

## 丢弃非最新的提交
用 `git rebase -i`


## 强制 push
```
git push origin branch1 -f
```
`-f` 是 `--force` 的缩写，意为「忽略冲突，强制 `push`」。

# 偏移符号

 Git 中，有两个「偏移符号」： `^` 和 `~`。

 `^` 的用法：在 `commit` 的后面加一个或多个 `^` 号，可以把 `commit` 往回偏移，偏移的数量是 `^` 的数量。例如：`master^` 表示 `master` 指向的 `commit` 之前的那个 `commi`t； `HEAD^^` 表示 `HEAD` 所指向的 `commit` 往前数两个 `commit`。

`~` 的用法：在 `commit` 的后面加上 `~` 号和一个数，可以把 `commit` 往回偏移，偏移的数量是 `~` 号后面的数。例如：`HEAD~5` 表示 `HEAD` 指向的 `commit` 往前数 `5` 个 `commit`。

# 临时存放工作目录的改动

存放

```
git stash
```

!> 注意：没有被 track 的文件（即从来没有被 add 过的文件不会被 stash 起来，因为 Git 会忽略它们。如果想把这些文件也一起 stash，可以加上 `-u` 参数，它是 `--include-untracked` 的简写。就像这样：`git stash -u`


取出

```
git stash pop
```

# .gitignore

排除不想被管理的文件和目录

语法

``` python
# 忽略 .a 文件
*.a
# 但否定忽略 lib.a, 尽管已经在前面忽略了 .a 文件
!lib.a
# 仅在当前目录下忽略 TODO 文件， 但不包括子目录下的 subdir/TODO
/TODO
# 忽略 build/ 文件夹下的所有文件
build/
# 忽略 doc/notes.txt, 不包括 doc/server/arch.txt
doc/*.txt
# 忽略所有的 .pdf 文件 在 doc/ directory 下的
doc/**/*.pdf
```

# 打标签

Git 使用两种主要类型的标签：轻量标签（lightweight）与附注标签（annotated）。

推荐使用附注标签


| 命令 | 说明 |
|:-------|:------
| `git tag` | 列出标签
| `git tag -l 'v1.8.5*'` | 列出指定标签
| `git tag -a v1.4 -m 'my version 1.4'` | 使用 `-a` **创建附注标签** <br> 用 `-m` 附带说明信息
| `git show v1.4` | 查看标签对应的提交信息
| `git push origin v1.5` | 推送指定标签
| `git push origin --tags` | 推送所有标签

!> `git push` 默认不会推送标签

# commit message

http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html


每次提交，Commit message 都包括三个部分：Header，Body 和 Footer。

```
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
```

其中，Header 是必需的，Body 和 Footer 可以省略。

不管是哪一个部分，任何一行都不得超过72个字符（或100个字符）。这是为了避免自动换行影响美观。

## Header

Header 部分只有一行，包括三个字段：type（必需）、scope（可选）和 subject（必需）。

type 用于说明 commit 的类别，只允许使用下面 7 个标识。

* feat：新功能（feature）
* fix：修补bug
* docs：文档（documentation）
* style： 格式（不影响代码运行的变动）
* refactor：重构（即不是新增功能，也不是修改bug的代码变动）
* test：增加测试
* chore：构建过程或辅助工具的变动


scope 用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

subject 是 commit 目的的简短描述，不超过 50 个字符。

* 以动词开头，使用第一人称现在时，比如 change，而不是 changed 或 changes
* 第一个字母小写
* 结尾不加句号（.）

# 配置 SSH

https://segmentfault.com/a/1190000005349818

https://ioliu.cn/2015/generating-ssh-key/

在给 github 配置 ssh 时如果 `~/.ssh/id_rsa` 文件已经存在并且其中的邮箱并不是 github 邮箱账号，那就没有办法复用，需要创建新的秘钥文件。

``` bash
ssh-keygen -t rsa -C "jianshijiuyou@gmail.com"
```

出现如下提示：

输入新路径，这里我输入的是 `/home/jianshijiuyou/.ssh/github`

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/jianshijiuyou/.ssh/id_rsa):/home/jianshijiuyou/.ssh/github
```

回车输入密码(可以不输入)

```
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/jianshijiuyou/.ssh/github.
Your public key has been saved in /home/jianshijiuyou/.ssh/github.pub.
The key fingerprint is:
SHA256:CxLQryHf1yCJtbC6ouYEqPZAml1EO69U++JMn+0kjvM jianshijiuyou@gmail.com
The key's randomart image is:
+---[RSA 2048]----+
|  ...            |
|   oo..          |
|    =*.o         |
|. ..+==..        |
|o. ++++.So       |
|=o.ooo.o...      |
|++... o.+ .      |
|+.+  +.= =       |
|=o .  +oE.o      |
+----[SHA256]-----+
```

然后

``` bash
vim ~/.ssh/config
```

输入

```
Host github.com
  User git
  IdentityFile /home/jianshijiuyou/.ssh/github
  IdentitiesOnly yes
```

然后将 `github.pub` 中的内容复制到 github 账户的 SSH key 配置中

?> 进入 GitHub 账户设置，点击左边 `SSH Key` ，点击 `Add SSH key` ，粘贴刚刚复制的内容，然后保存。

测试

``` bash
ssh -T git@GitHub.com
```

成功标识

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

## 如果是要用 SSH 免密登录服务器

客户端

```
 ~/.ssh  ssh-keygen -t rsa -C "root"   # 登录名
Generating public/private rsa key pair.
Enter file in which to save the key (/home/jianshijiuyou/.ssh/id_rsa): bwg # 随便起个名字，不要和现有的冲突
Enter passphrase (empty for no passphrase):   # 直接回车
Enter same passphrase again:  # 直接回车
Your identification has been saved in bwg.
Your public key has been saved in bwg.pub.
The key fingerprint is:
SHA256:lwEXruFndkpPvc7uGCE8mqw3xJ/pX38D5oE8BM4clYw root
The key's randomart image is:
+---[RSA 2048]----+
|        . *o.    |
|         E o     |
|        = =      |
|       . B + .   |
|       .S & = .  |
|       .oO @ = . |
|       .+..o* =  |
|       .o +  B o.|
|      .. o..oo= +|
+----[SHA256]-----+
 ~/.ssh  ls
bwg  bwg.pub
 ~/.ssh  scp -P 28957 bwg.pub root@xx.xxxx.com:~/.ssh # 将公钥拷贝到服务器
```

服务器

```
cat ~/.ssh/bwg.pub >> ~/.ssh/authorized_keys
```

客户端

```
ssh root@xx.xxxx.com -p 28957
```

(完)