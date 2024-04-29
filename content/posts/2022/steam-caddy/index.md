---
title: 在Linux上使用Caddy反代Steam社区
date: 2022-10-14 15:20:00
categories: 技术分享
tags: [Linux, Network]
index_img: /img/posts/steam-caddy.webp
banner_img: /img/posts/steam-caddy.webp
---

在 Linux 上 使用 Caddy 原生实现 [Steamcommunity 302](https://www.dogfight360.com/blog/686/) 的功能并实现开机自启

<!-- more -->

### 生成配置文件

为了生成自定义配置，需要在 Wine 下运行 Steamcommunity 302，在设置里勾选需要的服务，将证书期限修改为1年，点击 `导出SteamDeck版本一键安装脚本`

在 `SteamDeck_302` 文件夹下将如下几个文件到任意目录，这里假定为 `/data/caddy`，根据需要自行替换目录

> steamcommunity.crt
> 
> steamcommunity.key
> 
> steamcommunity_302.caddy.json
> 
> steamcommunityCA.pem

复制 Steamcommunity 302 设置里的 hosts 到 Linux 的 `/etc/hosts`，根据反代服务选择不同，hosts 条目也不同，请复制自己设置内的 hosts

```
127.0.0.1 steamcommunity.com
127.0.0.1 www.steamcommunity.com
127.0.0.1 store.steampowered.com
127.0.0.1 api.steampowered.com
127.0.0.1 help.steampowered.com
127.0.0.1 login.steampowered.com
127.0.0.1 store.akamai.steamstatic.com
```

### 下载 Caddy

在 [caddy/release](https://github.com/caddyserver/caddy/releases/tag/v2.6.4) 下载对应平台的 caddy（如 amd64 下载  `caddy_2.6.4_linux_amd64.tar.gz` ）

解压出来 `caddy` 二进制文件放入 `/data/caddy`

此时你的 `/data/caddy` 文件夹结构应如下

```
caddy
├── caddy
├── steamcommunity_302.caddy.json
├── steamcommunityCA.pem
├── steamcommunity.crt
└── steamcommunity.key
```

### 编辑配置

<details>

  <summary>使用SteamDeck版本一键安装脚本则无需修改</summary>

打开 `steamcommunity_302.caddy.json`

找到随机生成的端口号，此处端口为 19736

```
https://steamcommunity.com:19736 https://www.steamcommunity.com:19736 {
    tls steamcommunity.crt steamcommunity.key
    @steamcommunityrp {
    path /comment/*
    path /forum/*
···

https://store.steampowered.com:19736 https://api.steampowered.com:19736 https://help.steampowered.com:19736 https://login.steampowered.com:19736 https://store.akamai.steamstatic.com:19736 {
#tls self_signed
tls steamcommunity.crt steamcommunity.key
···
```

将所有的 `:19736` 删除，处理后的部分如下

```
https://steamcommunity.com https://www.steamcommunity.com {
    tls steamcommunity.crt steamcommunity.key
    @steamcommunityrp {
    path /comment/*
    path /forum/*
···

https://store.steampowered.com https://api.steampowered.com https://help.steampowered.com https://login.steampowered.com https://store.akamai.steamstatic.com {
#tls self_signed
tls steamcommunity.crt steamcommunity.key
···
```

</details>

### 导入根证书

Archlinux：
```
# cp steamcommunityCA.pem /etc/ca-certificates/trust-source/anchors/steamcommunityCA.crt

# trust extract-compat
```

Centos:
```
# cp steamcommunityCA.pem /etc/pki/ca-trust/source/anchors

# /bin/update-ca-trust
```
Ubuntu:
```
# cp steamcommunityCA.pem /usr/local/share/ca-certificates

# update-ca-certificates
```

> Centos 和 Ubuntu 的导入方式来自 [Dogfight360](https://www.dogfight360.com/blog/2319/) ，不保证一定可用

### 启动服务

赋予caddy可执行权限

```
chmod +x caddy
```

赋予caddy监听所需端口权限

```
sudo setcap CAP_NET_BIND_SERVICE=+eip caddy
```

启动

```
./caddy run --config steamcommunity_302.caddy.json --adapter caddyfile
```

如启动无报错 Ctrl+C 强行停止，配置 Systemd 服务

新建编辑 `.config/systemd/user/caddy.service`，修改两处 `/data/caddy`

```
[Unit]
Description=Caddy Steam Reverse Proxy

[Service]
WorkingDirectory=/data/caddy
ExecStart=/data/caddy/caddy run --config steamcommunity_302.caddy.json --adapter caddyfile

[Install]
WantedBy=default.target
```

运行并启动服务

```
$ systemctl --user daemon-reload
$ systemctl --user enable --now caddy.service
```

查看服务状态

```
$ systemctl --user status caddy.service
```

### 疑难解答

- steamcommunity_302.caddy.json 不存在？
  steamcommunity 302 未成功运行，尝试在 windows 环境下重新运行
