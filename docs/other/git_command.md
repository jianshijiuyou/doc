> 摘要自：[https://juejin.im/book/5a124b29f265da431d3c472e](https://juejin.im/book/5a124b29f265da431d3c472e)


# 常用 Git 命令
| 命令 | 说明
|:-----------
| `git clone <URL>` | 获取远程仓库的文件到本地
| `git init` | 在本地初始化仓库
| `git remote add <origin> <URL>` | 添加远程仓库
| `git status` | 查看工作目录当前状态
| `git add <filename>` | 添加要跟踪的文件 <br> 常用: `git add .`
| `git commit [-m "log"]` | 提交，`-m` 后跟提交时附带的日志
| `git log` | 查看提交历史 <br> 查看详细历史：`git log -p` <br> 简要统计：`git log --stat`
| `git show [commit id]` | 查看 commit 的具体改动，默认查看当前 commit
| `git push [仓库名] [分支名]` | 把本地的提交记录推送到服务器 <br> 常用: `git push -u origin master` <br> 不指定默认推送 `master` 分支 <br> 有时需要指定其他分支: `git push origin newBranch`
| `git pull` | 把服务器提交记录拉取到本地
| `git branch <branch name>` | 创建分支
| `git checkout <branch name>` | 切换分支
| `git checkout -b <branch name>` | 创建分支同时切换到该分支上
| `git branch -d <branch name>` | 删除分支，<br>执行完后再执行 `git push origin -d <branch name>`, <br>可以把服务器端的分支也删除了
| `git merge <branch name>` | 把当前分支和指定分支合并 <br> 一般在 master 分支上操作，合并其他分支
| `git merge --abort` | 取消合并
| `git rebase master` | 把当前分支所有的 commit 提交到 master 分支上 <br> 一般执行还需要执行 `git checkout master` 和 `git merge branch1` <br> **一般不会在 master 分支上执行 `rebase`**
| `git config --global core.editor vim` | 将默认 commit 的编辑器修改为 vim |

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
|:-------------
| `git tag` | 列出标签
| `git tag -l 'v1.8.5*'` | 列出指定标签
| `git tag -a v1.4 -m 'my version 1.4'` | 使用 `-a` **创建附注标签** <br> 用 `-m` 附带说明信息
| `git show v1.4` | 查看标签对应的提交信息
| `git push origin v1.5` | 推送指定标签
| `git push origin --tags` | 推送所有标签

!> `git push` 默认不会推送标签