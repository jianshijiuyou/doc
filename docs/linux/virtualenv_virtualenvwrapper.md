## virtualenv + virtualenvwrapper

## 安装 virtualenv

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

## 安装 virtualenvwrapper

```
sudo pip install virtualenvwrapper
```

在 `.bashrc` (或 `.zshrc` ) 中添加下列内容：

```
export WORKON_HOME=$HOME/.virtualenvs
source /usr/local/bin/virtualenvwrapper.sh
```

!> virtualenvwrapper.sh 文件可能在其他地方

使修改生效

```
source ~/.bashrc # (或~/.zshrc)。
```
### 使用
 * `workon`: 打印所有的虚拟环境；  
 * `mkvirtualenv xxx`: 创建 xxx 虚拟环境;  
 * `workon xxx`: 使用 xxx 虚拟环境;  
 * `deactivate`: 退出 xxx 虚拟环境；  
 * `rmvirtualenv xxx`: 删除 xxx 虚拟环境。  