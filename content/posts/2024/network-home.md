---
date: "2024-05-05T20:50:49+08:00"
title: "安全便利地访问家里的 NAS"
tags: [Linux, Network]
categories: 技术分享
---

通过代理访问家中拥有公网 IP 的 NAS

<!-- more -->

## 前言

众所周知，直接在拥有公网 IP 的家宽上开放 80,443 等端口是可能会被运营商查水表的。但当用户拥有大量内网服务时，记端口是个大问题，我们需要一种能直接通过域名访问内网服务，且出门在外时使用手机也能访问到内网服务的方案

因此，我们将使用 ShadowSocks 作为连回内网的桥梁，使用 Caddy 为内网服务提供反向代理。这种方式有以下几个优点：

- 无感：在实际使用中用户直接使用域名访问服务
- 快速：与 frp 转发等方案相比，用户与服务直连，不会被云服务器带宽卡脖子
- 扩展性高：添加了新服务后只需修改 Caddy 配置即可立即使用

但是同时也带来了一些问题：

- 配置稍加繁琐：用户端需要配置 Clash 等能够链接 ShadowSocks 的软件
- 硬性要求公网 IP

## 要求

- 具有公网 IP 的家宽
- 接入宽带的 Linux 设备
- 可以使用 Clash 接入网络的终端
- 支持 DDNS 及通配符解析的域名

## 配置

### 准备

首先确定需要访问的服务的 IP 及端口，例如在内网中运行的 EMBY Server、Qbittorrent、PVE 等服务，接下来将会以这些服务作为例子

- EMBY Server: 192.168.50.254:8096
- Qbittorrent: 192.168.50.254:8081
- PVE: 192.168.50.10:8006

本例中将使用域名 `example.com` 作为例子进行域名配置，域名托管在 cloudflare 上

### 配置 DNS

1. 使用 DDNS 解析工具进行家宽 IP 的固定解析，例如 [ddns-go](https://github.com/jeessy2/ddns-go)，将 IP 解析到 `a.example.com` 上，具体操作步骤参考 ddns 工具文档

2. 将 `*.a.example.com` 解析到 Caddy 所在的地址上，例如 `192.168.50.254`

3. 获取 cloudflare 的 API Token

   进入 [API Tokens](https://dash.cloudflare.com/profile/api-tokens) 页面，点击 `Create Token` 按钮，再点击 `Get started`，填写 `Token name`，在 `Permissions` 中添加两个权限：

   - Zone / Zone / Read
   - Zone / DNS / Edit

   配置完成后记下 Token，配置 Caddy 时需要用到

### 配置 Caddy

在内网中的服务器上配置 Caddy 服务，最好在上述的 EMBY 等流量可能会很大的服务的同一台服务器上配置，同时需要此服务器的 `80,443` 端口可供内网访问，但无需对外网开放。将要配置 Caddy 的服务器的 IP 例为 `192.168.50.254`

#### 编译镜像

由于本文使用 Cloudflare 托管域名，需要使用具有 Cloudflare 模块的 Caddy 进行证书的配置，首先编写 Dockerfile：

```bash
FROM caddy:2.7-builder AS builder

RUN xcaddy build \
    --with github.com/caddy-dns/cloudflare

FROM caddy:2.7

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

在 Dockerfile 所在的目录下运行 `docker build --tag=caddy-cf .` 编译镜像

#### 配置 Caddyfile

根据自己服务填写 Caddyfile，将上面得到的 `cloudflare API Token` 填入配置中

```Caddyfile
{
  http_port 80
  https_port 443
}
*.a.example.com {
  tls {
    dns cloudflare YOUR_API_TOKEN_HERE
  }
}

https://emby.a.example.com {
  reverse_proxy 192.168.50.254:8096
}

https://qbit.a.example.com {
  reverse_proxy 192.168.50.254:8081
}

https://pve.a.example.com {
        reverse_proxy 192.168.50.10:8006 {
                transport http {
                        tls_insecure_skip_verify
                }
        }
}
```

#### 启动 Caddy

完成后启动 Caddy，注意需要 `--cap-add=NET_ADMIN`，以及 `80,443` 端口未被其他应用占用

```bash
docker run -d --cap-add=NET_ADMIN --name=caddy --restart=always \
  -v "$PWD"/Caddyfile:/etc/caddy/Caddyfile \
  -v "$PWD"/data:/data \
  --network=host \
  caddy-cf
```

完成后此时使用内网设备应该已经能使用 `https://emby.a.example.com` 等在 Caddy 中配置过的域名访问对应服务了，但是因为 `*.a.example.com` 解析到了一个内网地址，所以无法从公网访问

### 配置 ShadowSocks

和 Caddy 同理，因为流量会经过 ShadowSocks 进行转发，所以尽量将 ShadowSocks 安装在大流量服务的同一服务器上

安装 [shadowsocks-rust](https://github.com/shadowsocks/shadowsocks-rust)

在 `/etc/shadowsocks-rust/server.json` 中填写以下内容

```json
{
  "server": "0.0.0.0",
  "server_port": 51888,
  "password": "YOUR_PASSWORD_HERE",
  "timeout": 300,
  "method": "chacha20-ietf-poly1305",
  "dns": "google"
}
```

> 建议使用 `ssservice genkey -m "chacha20-ietf-poly1305"` 生成较强的密码

启动 ShadowSocks 服务 `systemctl enable --now shadowsocks-rust-server@server.service`

ShadowSocks 所需要的端口可以任意指定，但需要在后面的步骤中配置端口转发且暴露至公网，建议使用随机高端口号以及较强的密码保证安全

### 端口转发

在内网主路由器上配置端口转发，配置方法根据路由品牌各有不同。

以华硕为例，进入 `外部网络-端口转发-添加设置文件`

- 外部端口填写任意端口，后续配置 Clash 时需要这个端口以连接 ShadowSocks 服务
- 内部端口填写上述的 ShadowSocks 配置文件中的 `server_port`
- 本地 IP 地址填写 SHadowSocks 服务所在的服务器内网 IP

本例中为

| 外部端口 | 内部端口 | 本地 IP 地址   | 通信协议 |
| -------- | -------- | -------------- | -------- |
| 51888    | 51888    | 192.168.50.254 | BOTH     |

这样理论上就能通过 `路由器公网IP:51888` 端口访问 ShadowSocks 服务了；由于我们已经配置了 DDNS，所以也能通过 `a.example.com:51888` 端口访问 ShadowSocks 服务

### 配置 Clash

在需要在外网访问内网的设备上配置 Clash 客户端

在 Clash 配置文件中添加如下配置

```yaml
proxies:
  - name: "NAS"
    type: ss
    server: a.example.com
    port: 51888
    cipher: chacha20-ietf-poly1305
    password: "YOUR_PASSWORD_HERE"

rules:
  - DOMAIN-SUFFIX,a.example.com,NAS
```

或者如果想要在内网环境下不流量经过 ShadowSocks，直接使用直连，可以使用如下配置

```yaml
proxies:
  - name: "NAS"
    type: ss
    server: a.example.com
    port: 51888
    cipher: chacha20-ietf-poly1305
    password: "YOUR_PASSWORD_HERE"

proxy-groups:
  - {
      name: HOME,
      type: fallback,
      proxies: [DIRECT, NAS],
      url: "https://test.a.example.com",
      interval: 3600,
    }

rules:
  - DOMAIN-SUFFIX,a.example.com,HOME
```

内网下 `test.a.example.com` 指向 `192.168.50.254`，url-test 时连通，所以 `a.example.com` 会走 `DIRECT` 直连

外网环境下不连通，`a.example.com` 的流量会分流到 `NAS` 规则下，经过 ShadowSocks 转发至内网中

### 测试

至此配置完成，使用终端测试连接，在开启了 Clash 后，访问 `emby.a.example.com` 应能正常连接，且拥有完整的 TLS 加密
