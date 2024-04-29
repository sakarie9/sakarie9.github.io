---
title: Telegram 收发 QQ 信息 - EFB 和 GO-CQHTTP 的 Docker 部署教程
date: 2021-11-15 18:13:39
categories: 技术分享
tags: [Linux, Docker, Telegram]
index_img: /img/posts/tg-qq.webp
banner_img: /img/posts/tg-qq.webp
---

使用 Bot 在 Telegram 及 QQ 间转发消息，基本可以做到去 QQ 化，仅在 Telegram 上与 QQ 好友/群组互动

<!-- more -->

## 简介

使用 Bot 在 Telegram 及 QQ 间转发消息，基本可以做到去 QQ 化，仅在 Telegram 上与 QQ 好友/群组互动

本项目使用 Docker Compose 简化了 [Telegram Bot](https://github.com/ehForwarderBot/ehForwarderBot) 和 [QQ Bot](https://github.com/Mrs4s/go-cqhttp) 的安装与配置，仅需要 Docker Compose 与流畅的国际互联网连接即可使用

### 环境要求

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- 能稳定连接到Telegram服务器的网络

### 使用项目

- [TG-EFB-QQ-Docker](https://github.com/sakarie9/TG-EFB-QQ-Docker)

  - [EFB-Docker](https://github.com/sakarie9/EFB-Docker)
    - [EH Forwarder Bot](https://github.com/ehForwarderBot/ehForwarderBot)
    - [Telegram Master](https://github.com/ehForwarderBot/efb-telegram-master)
    - [QQ Slave](https://github.com/milkice233/efb-qq-slave)
    - [QQ Plugin Mirai](https://github.com/milkice233/efb-qq-plugin-mirai)
    - [QQ Plugin go-cqhttp](https://github.com/XYenon/efb-qq-plugin-go-cqhttp)
    - [efb-filter-middleware](https://github.com/sakarie9/efb-filter-middleware)

  - [Go-CQHttp-Docker](https://github.com/sakarie9/Go-CQHTTP-Docker)
    - [go-cqhttp](https://github.com/Mrs4s/go-cqhttp)

## 配置

### 克隆项目

```bash
# 克隆
git clone -b go-cqhttp https://github.com/sakarie9/TG-EFB-QQ-Docker.git
# 进入文件夹
cd TG-EFB-QQ-Docker
```

### 配置`GOCQ`端

1. 编辑 `gocq/config.yml` 配置文件

    ```bash
    account:         # 账号相关
      uin: 000000000 # QQ 账号
      password: ''   # QQ 密码，为空时使用扫码登录
    ```

2. （可选）修改登陆协议，运行如下命令，待提示生成 `device.json` 后 `ctrl+c` 退出，编辑 `gocq/device.json`，参考 [设备信息](https://docs.go-cqhttp.org/guide/config.html#%E8%AE%BE%E5%A4%87%E4%BF%A1%E6%81%AF)

    ```bash
    docker run --rm -it --name="gocq" -v $PWD/gocq:/data xzsk2/gocqhttp-docker:latest
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


## 运行

### 启动

```bash
docker-compose up -d
```

如需扫码登陆输入 `docker logs gocq` 查看二维码

### 停止

```bash
docker-compose down
```

### 自动更新

```
docker run -d \
    --name watchtower \
    --restart unless-stopped \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower -c \
    --interval 3600 \
    efb gocq
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

3. 修改`EFB`配置

    编辑`./TG-EFB-QQ-Docker/efb/profile/default/blueset.telegram/config.yaml`，添加代理

    ```yaml
    token: xxx:xxx
    admins:
    - xxxxxxxx
    # 添加下面的两行
    request_kwargs:
        proxy_url: socks5h://127.0.0.1:1080/
    ```

4. 重启

    ```bash
    docker-compose down
    docker-compose up -d
    ```

## 常见问题

- 无法发送群消息，只能接收？

    无法发送大于三个字符的群消息，接收正常；好友消息的收发正常。这种情况是触发了TX的风控，一般服务器上挂12小时-2天即可正常
