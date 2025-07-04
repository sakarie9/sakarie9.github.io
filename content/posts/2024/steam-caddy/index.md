---
title: 在 Linux 上反代 Steam 社区
date: 2024-05-07
categories: 技术分享
tags:
  - Linux
  - Network
summary: 在 Linux 上 使用 Steamcommunity 302 并使用 systemd 实现开机自启
---

在 Linux 上 使用 [Steamcommunity 302](https://www.dogfight360.com/blog/18682/) 并使用 systemd 实现开机自启

<!-- more -->

### 配置

根据需要勾选所需的功能

```bash
chmod +x Steamcommunity_302
./Steamcommunity_302
```

![steam302.webp](steam302.webp)

### 测试

```bash
chmod +x steamcommunity_302.cli
sudo ./steamcommunity_302.cli
```

测试是否能正常访问 steam 及社区

### 自启

修改路径

```plain
# /etc/systemd/system/steamcommunity-302.service

[Unit]
Description=Steamcommunity_302 Steam Reverse Proxy
After=network-online.target
Wants=network-online.target

[Service]
WorkingDirectory=/path_to_302/Steamcommunity_302
ExecStart=/path_to_302/Steamcommunity_302/steamcommunity_302.cli

[Install]
WantedBy=multi-user.target
```

然后启动

```bash
systemctl enable --now steamcommunity-302.service
```

---

{{< alert >}}

v13 已经原生支持 Linux，以下为旧版本配置

{{< /alert >}}

在 Linux 上 使用 Caddy 原生实现 [Steamcommunity 302](https://www.dogfight360.com/blog/686/) 的功能并实现开机自启

{{< alert >}}

以下操作均在 Arch Linux 下进行，其他发行版可能需要适当修改证书安装的路径及所用命令

{{< /alert >}}

### 生成配置文件

为了生成自定义配置，需要在 Wine 下运行 Steamcommunity302，在设置里勾选需要的服务，将证书期限修改为 10 年，点击 `导出SteamDeck版本一键安装脚本`

在 `SteamDeck_302` 文件夹下将如下几个文件到任意目录，这里假定为 `/data/caddy`，根据需要自行替换目录

- steamcommunity.crt
- steamcommunity.key
- steamcommunity_302.caddy.json
- steamcommunityCA.pem

复制 Steamcommunity 302 设置里的 hosts 到 Linux 的 `/etc/hosts`，根据反代服务选择不同，hosts 条目也不同，请复制自己设置内的 hosts

```hosts
127.0.0.1 steamcommunity.com #S302
127.0.0.1 www.steamcommunity.com #S302
127.0.0.1 store.steampowered.com #S302
127.0.0.1 checkout.steampowered.com #S302
127.0.0.1 api.steampowered.com #S302
127.0.0.1 help.steampowered.com #S302
127.0.0.1 login.steampowered.com #S302
127.0.0.1 store.akamai.steamstatic.com #S302
```

### 下载 Caddy

在 [caddy/release](https://github.com/caddyserver/caddy/releases/tag/v2.7.6) 下载对应平台的 caddy（如 amd64 下载 `caddy_2.7.6_linux_amd64.tar.gz` ）

解压出来 `caddy` 二进制文件放入 `/data/caddy`

此时你的 `/data/caddy` 文件夹结构应如下

```plain
caddy
├── caddy
├── steamcommunity_302.caddy.json
├── steamcommunityCA.pem
├── steamcommunity.crt
└── steamcommunity.key
```

### 编辑配置

<details>

  <summary>目前使用SteamDeck版本一键安装脚本则无需修改，请略过本节</summary>

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

### 安装证书

```bash
certutil -A -d sql:~/.pki/nssdb -n "Steamcommunity302" -t C,, -i "steamcommunityCA.pem"

sudo cp steamcommunityCA.pem /etc/ca-certificates/trust-source/anchors/steamcommunityCA.crt

sudo trust extract-compat
```

### 启动服务

赋予 caddy 可执行权限

```bash
chmod +x caddy
```

赋予 caddy 监听所需端口权限

```bash
sudo setcap CAP_NET_BIND_SERVICE=+eip caddy
```

启动

```bash
./caddy run --config steamcommunity_302.caddy.json --adapter caddyfile
```

如启动无报错 Ctrl+C 强行停止，配置 Systemd 服务

新建编辑 `.config/systemd/user/caddy.service`，修改两处 `/data/caddy`

```bash
[Unit]
Description=Caddy Steam Reverse Proxy

[Service]
WorkingDirectory=/data/caddy
ExecStart=/data/caddy/caddy run --config steamcommunity_302.caddy.json --adapter caddyfile

[Install]
WantedBy=default.target
```

运行并启动服务

```bash
systemctl --user daemon-reload
systemctl --user enable --now caddy.service
```

查看服务状态

```bash
systemctl --user status caddy.service
```

### 疑难解答

- steamcommunity_302.caddy.json 不存在？
  steamcommunity 302 未成功运行，尝试在 windows 环境下重新运行
