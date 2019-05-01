# 安装

``` bash
npm install -g pm2
```

更新

``` bash
npm install pm2 -g && pm2 update
```

# 基本用法

| 命令 | 说明 |
|:-----|:------
| `pm2 start app.js` | 启动并添加一个进程到 pm2 的进程管理列表。 <br> 默认进程名是文件名（不带后缀）<br> `--name` 或 `-n` 指定进程名。<br> 例: `pm2 start main.py -n www` 
| `pm2 ls` | 显示列表
| `pm2 delete app` | 删除进程
| `pm2 stop app` | 停止
| `pm2 restart app` | 重启
| `pm2 logs app` | 查看日志 <br> 日志文件都在 `~/.pm2/logs` 目录下
| `pm2 reload app` | 重新加载
| `pm2 init` | 生成配置文件模板 `ecosystem.config.js`
| `pm2 save` | 保存进程列表
| `pm2 resurrect` | 恢复进程列表 （从 save 保存的文件中恢复并启动进程）
| `pm2 startup` | 生成一条服务器再重启后自动启动 pm2 和它维护的进程的命令,这条命令需要手动执行。<br>当执行了 `pm2 save` 后，再执行 `pm2 startup` 生成的命令，就可以在服务器重启后自动重启 pm2 维护的进程。<br>**要取消这个操作可以删除 pm2 save 生成的 dump 文件**
| `pm2 monit` | 打开监控
| `pm2 logs` | 所有 log
| `pm2 logs app` | 指定进程 log
| `pm2 flush` | 清空所有进程 log

# 查看端口号

根据 `pm2 ls` 中显示的 `pid` 查看服务对应的端口号

``` bash
ss -altunp|grep [pid]
# or
ps aux|grep [pid]
```

# Ecosystem 配置文件

`pm2 init` 生成配置文件模板 `ecosystem.config.js`

`pm2 start` 默认会找当前目录下的 `ecosystem.config.js` 文件进行进程的加载启动

指定配置文件

``` bash
pm2 start /path/to/ecosystem.config.js
```

# 环境变量

指定环境变量

``` bash
pm2 start ecosystem.config.js --env production
```

更新环境变量

``` bash
pm2 restart ecosystem.config.js --update-env
```

切换环境变量

``` bash
pm2 restart ecosystem.config.js --env production --update-env
```

# 脚本类型

pm2 默认支持以下脚本

``` javascript
{
  ".sh": "bash",
  ".py": "python",
  ".rb": "ruby",
  ".coffee": "coffee",
  ".php": "php",
  ".pl": "perl",
  ".js": "node"
}
```

?> 没有扩展名，应用程序将作为二进制文件启动。

例如，要在python中启动脚本，请使用：

``` bash
pm2 start echo.py
```

如果要指定解释器的路径，请在 ecosystem 文件中指定它：

``` javascript
module.exports = {
  "apps" : [{
    name: "script",
    script: "./script.py",
    interpreter: "/usr/bin/python",
  }]
}
```

# 指定日志位置

``` javascript
module.exports = {
  apps: [{
      name: 'app',
      script: 'app.js',
      output: './out.log',
      error: './error.log',
	    log: './combined.outerr.log',
    }]
}
```

- `output` 只保存标准输出 (console.log)
- `error` 只保存错误输出 (console.error)
- `log` 都保存, 默认禁用