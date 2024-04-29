---
title: （旧）Telegram 收发 QQ 信息 - EFB 和 Mirai 的 Docker 部署教程
date: 2021-05-05 17:13:39
categories: Archived
tags: [Linux, Docker, Telegram]
index_img: /img/posts/tg-qq.webp
banner_img: /img/posts/tg-qq.webp
---

使用 Bot 在 Telegram 及 QQ 间转发消息，基本可以做到去 QQ 化，仅在 Telegram 上与 QQ 好友/群组互动

<!-- more -->

## 简介

使用 Bot 在 Telegram 及 QQ 间转发消息，基本可以做到去 QQ 化，仅在 Telegram 上与 QQ 好友/群组互动

本项目使用 Docker Compose 简化了 [Telegram Bot](https://github.com/ehForwarderBot/ehForwarderBot) 和 [QQ Bot](https://github.com/Mrs4s/go-cqhttp) 的安装与配置，仅需要 Docker Compose 与流畅的国际互联网连接即可使用

本教程为 Mirai 版本，推荐使用新版

{{< article link="/posts/2021/tg-qq-gocq/" >}}

### 环境要求

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- 能稳定连接到Telegram服务器的网络

### 使用项目

- [TG-EFB-QQ-Docker](https://github.com/xzsk2/TG-EFB-QQ-Docker)

  - [EFB-Docker](https://github.com/xzsk2/EFB-Docker)
    - [EH Forwarder Bot](https://github.com/ehForwarderBot/ehForwarderBot)
    - [Telegram Master](https://github.com/ehForwarderBot/efb-telegram-master)
    - [QQ Slave](https://github.com/milkice233/efb-qq-slave)
    - [QQ Plugin Mirai](https://github.com/milkice233/efb-qq-plugin-mirai)

  - [Mirai-Docker](https://github.com/xzsk2/Mirai-Docker)
    - [Mirai Console Loader](https://github.com/iTXTech/mirai-console-loader)
    - [mirai-api-http](https://github.com/project-mirai/mirai-api-http)

## 配置

### 克隆项目

```bash
# 克隆
git clone -b mirai https://github.com/xzsk2/TG-EFB-QQ-Docker.git
# 进入文件夹
cd TG-EFB-QQ-Docker
```

### 配置`Mirai`端

1. 编辑`./mirai/config/net.mamoe.mirai-api-http/setting.yml`，修改`authKey`，最少8位，记下这个`authKey`，会在下面配置EFB的过程中用到

    ```yaml
    authKey: xxxxxxxx
    ```

### 配置`EFB`端

1. 获取`token`

    创建一个Bot，向 [@BotFather](https://t.me/BotFather) 发起会话，发送指令 `/newbot` 开始创建Bot，创建完成后可获取`token`

2. 查看自己的`Telegram ID`

    向 [@get_id_bot](https://t.me/get_id_bot) 发送`/start`，得到的`Chat ID`即为用户的`Telegram ID`

3. 打开`./efb/profiles/default/blueset.telegram/config.yaml`，修改下列字段，`token`修改为上面获取到的`Bot token`，`admins`修改为`Telegram ID`，注意格式

    ```yaml
    token: 123456789:ABCDEFG1ABCDEFG1ABCDEFG1
    admins:
    - 987654321
    ```

4. 打开`./efb/profiles/default/milkice.qq/config.yaml`，修改`qq`和`authKey`

    ```yaml
    Client: mirai
    mirai:
        qq: xxxxxxxx           # 这里换成登录的 QQ 号
        host: "mirai"       # 这个不要改
        port: 8080              # 同上
        authKey: "xxxxxxxx" # 这里填入在配置 Mirai API HTTP 时生成的 authKey
    ```

## 运行

1. 登录QQ，在`docker-compose.yml`文件夹下运行

    ```bash
    docker run --rm -it --name="mirai" -p 8080:8080 -v $PWD/mirai/config:/app/config -v $PWD/mirai/bots:/app/bots xzsk2/mirai-docker:latest
    ```

2. 使用`/login`登录账号

    ```bash
    /login <qq> <password>    # 登录一个账号
    ```

3. 出现`Login successful`后登录成功，再次运行

    ```bash
    docker run --rm -it --name="mirai" -p 8080:8080 -v $PWD/mirai/config:/app/config -v $PWD/mirai/bots:/app/bots xzsk2/mirai-docker:latest
    ```

4. 使用`/autologin`设置自动登录

    ```bash
    /autoLogin add <account> <password> # 添加自动登录
    ```

5. 之后使用`CTRL+C`退出容器，使用`docker-compose`启动

    ```bash
    docker-compose up -d
    ```

## 本地代理

如果你的服务器环境可以连接到Telegram服务器，可跳过本章节

本教程使用 [ssr-command-client](https://github.com/TyrantLucifer/ssr-command-client) 作为本地代理，可参考此项目文档配置

1. 安装

    ```bash
    pip3 install shadowsocksr-cli
    ```

2. 使用

    ```bash
    # 添加订阅链接
    shadowsocksr-cli --add-url 你的ssr订阅链接
    # 更新订阅
    shadowsocksr-cli -u
    # 启动
    shadowsocksr-cli --fast-node
	# 修改监听地址
	shadowsocksr-cli --setting-address 0.0.0.0
    ```

3. 修改`EFB`配置

    编辑`./TG-EFB-QQ-Docker/efb/profile/default/blueset.telegram/config.yaml`，添加代理

    ```yaml
    token: xxx:xxx
    admins:
    - xxxxxxxx
    # 添加下面的两行
    request_kwargs:
        proxy_url: socks5h://172.17.0.1:1080/
    ```

4. 重启

    ```bash
    docker-compose down
    docker-compose up -d
    ```

## 常见问题

- Mirai登陆失败/验证码/版本过低？

    请尝试先在本地桌面环境下部署Mirai，如需要滑动验证码请使用 [mirai-login-solver-selenium](https://github.com/project-mirai/mirai-login-solver-selenium)，或修改登录协议

- 无法发送群消息，只能接收？

    无法发送大于三个字符的群消息，接收正常；好友消息的收发正常。这种情况是触发了TX的风控，一般服务器上挂12小时-2天即可正常
