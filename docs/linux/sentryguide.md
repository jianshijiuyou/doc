# 安装

使用 docker 安装

> github: [https://github.com/getsentry/onpremise](https://github.com/getsentry/onpremise)

首先 clone 下上面的项目

```
git clone https://github.com/getsentry/onpremise.git
```

假设你刚刚克隆了这个存储库，以下步骤将会立即启动并运行！

可能需要对包含的 `docker-compose.yml` 文件进行修改以适应您的需求或您的环境。这些说明是您通常应该做的准则。

1. `mkdir -p data/{sentry,postgres}`  

    制作我们的本地数据库和 sentry 配置目录。这个目录是用 postgres 绑定的，所以你不会失去状态！

2. `docker-compose run --rm web config generate-secret-key`  

    生成一个密钥。将其作为 `SENTRY_SECRET_KEY` 添加到 `docker-compose.yml` 中。

3. `docker-compose run --rm web upgrade`

    建立数据库。使用交互式提示创建用户帐户。

4. `docker-compose up -d`

    启动所有服务（分离/背景模式）。

5. 在 `localhost:9000` 上访问你的实例！

# 邮件配置

修改 `docker-compose.yml` 文件

```
............
services:
  base:
    restart: unless-stopped
    build: .
    environment:
      # Run `docker-compose run web config generate-secret-key`
      # to get the SENTRY_SECRET_KEY value.
      SENTRY_SECRET_KEY: '#(#7_0p+o5&-)misr=jo!t5ee3^ya%e5-*o!7j)=j!74to#)oe'
      SENTRY_MEMCACHED_HOST: memcached
      SENTRY_REDIS_HOST: redis
      SENTRY_POSTGRES_HOST: postgres
      SENTRY_SERVER_EMAIL: '1052603396@qq.com'  # 这里
      SENTRY_EMAIL_HOST: 'smtp.qq.com'          # 这里
      SENTRY_EMAIL_USER: '1052603396@qq.com'    # 这里
      SENTRY_EMAIL_PASSWORD: 'xxxxxxx'          # 这里
      SENTRY_EMAIL_USE_TLS: 'True'              # 这里, Sentry 目前不支持 SSL
    volumes:
      - ./data/sentry:/var/lib/sentry/files
............
```

然后重新运行

```
sudo docker-compose up -d
```

!> 在 sentry 界面的『左下角 -> 管理 -> 邮件』中可以发送测试邮件

然后在『左下角 -> 账户 -> Email』中验证邮箱

验证通过后就可以在项目中设置邮件通知了